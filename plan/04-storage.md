# 4. Storage

## 4.1 Principle

| store | holds | why |
|---|---|---|
| **R2** (objects) | the documents themselves: one JSON document per thing (upload, problem, session, the student's map, profile, map check, bank item, probe list, cached lesson, the shared co-use counts), plus the photos | whole documents are read and written by key; R2 is the truth |
| **D1** (SQL) | small index rows only: who owns which problem, the same problem by fingerprint, a user's session list, accounts and guest links, daily counters, the problem bank index | fast lookups and counts; billed by rows read and written, so it stays small |

Every D1 row except the daily counters, the accounts and the guest links can be **rebuilt from R2** by one script. A D1 row only points to a document; when they disagree, the document wins.

Cost rules:
- A request that only shows a page reads nothing from D1 or R2.
- Every D1 operation is one statement or one batch, touches as few rows as it can, and never reads more rows than it returns.
- Values are always bound; SQL is never built from input text.

## 4.2 D1 tables

| table | one row means | main columns | keys and indexes |
|---|---|---|---|
| problems | a problem exists and who owns it | problem id; fingerprint of its text; owner (empty for a shared problem); when made | key: problem id; indexed by fingerprint and by owner |
| sessions | a diagnosis session in a user's list | session id; owner; problem; status (active, completed, abandoned); last change | key: session id; indexed by owner and last change |
| accounts | a Google account | account id; Google subject (unique); when made | key: account id |
| guest links | a guest browser that signed in to an account | guest id; account id | key: guest id (one account per guest; the first link wins) |
| daily usage | one day's count of one kind of work | scope (user, network, whole app); key (user id, hashed network address, or "all"); UTC day; kind (upload, AI call); count; cost | key: scope + key + day + kind |
| bank | a composed problem in the shared bank | item id; bank concept key; language; when made | key: item id; indexed by concept key and language |

Behaviour that matters to the product:
- Adding a problem or a bank row is safe to repeat: an existing id stays as it is.
- A fingerprint lookup returns at most a few problem ids; a user's session list is newest first with a small default length; the bank lookup returns a bounded list oldest first.
- Saving a session row inserts a new id or updates only status and time.
- Creating an account is atomic: one Google subject is always one account.
- Taking one unit of a daily limit checks and counts in one atomic step, so two requests at the same time can never both pass the limit.
- Deleting a user removes their sessions, problems, guest links (as guest or as account), the daily-usage rows keyed by the user and the account row; other users' rows stay.

## 4.3 R2 documents and the key scheme

Every id inside a key must match letters, digits, `_` and `-`, 1 to 64 characters. Any other id is refused before a key is built, so user input can never reach another key.

| key | holds | written by |
|---|---|---|
| `photos/<upload id>` | the uploaded photo bytes with their content type | uploads |
| `users/<user id>/uploads/<upload id>.json` | the Upload record | uploads |
| `problems/<problem id>.json` | the ProblemDoc: the problem and its solution graph (empty until split) | uploads, problems |
| `users/<user id>/sessions/<session id>.json` | the Session record | sessions |
| `users/<user id>/map.json` | the MapDoc: events, the knowledge record (the evidence the map is built from), demo clock offset, constellations | map store |
| `users/<user id>/profile.json` | the signed-in email | sign-in |
| `users/<user id>/checks/<check id>.json` | the MapCheck record | map check |
| `bank/<bank concept key>/<language>/<item id>.json` | a ComposedItem | the map check, after the composer |
| `probes/<concept id>.<language>.json` | the list of probes for one concept and language | content |
| `cache/teach/<concept id>.<language>.json` | the TeachCard of a concept | content |
| `cache/teach/<concept id>#<pitfall id>.<language>.json` | the TeachCard that clears one pitfall of a concept | content |
| `co-use.json` | the shared co-use counts (main → support → count) | co-use counts |

The **bank concept key** is the main concept id plus the sorted support ids. The same value is the `bank` row's concept key. Every id inside it must meet the id rule.

Everything a user owns lives under `users/<user id>/`, plus the problem documents they own and their photos. This makes moving and deleting a user simple.

Documents are stored as JSON text with the JSON content type. Photos keep their own content type. A photo is kept only until 24 hours after it was read; the photo sweep (4.6) removes it, and "Delete my data" removes it at once.

## 4.4 Id prefixes

| id | form |
|---|---|
| guest user | `gst-` + random |
| account | `acc-` + random |
| upload, and the problem made from it | `upl-` + random (the problem keeps the upload's id) |
| session | `ses-` + random |
| composed bank item | `cmp-` + random |
| probe | `prb-` + random |
| map event record | made from the session or check it came from; unique per source |

Random parts are long enough to be unguessable and always fit the id rule.

## 4.5 Safe concurrent writes

Two requests can change the same document at once (two taps on Send, two tabs, a background job and a request). Neither may silently win, or a student's answer could be lost.

- Every change to a JSON document is a **versioned update**: read the document and its version tag, compute the new value, and write only if the stored version is still the one that was read (create-only when there was none).
- A lost write starts again from a fresh read, a few times. If it still cannot write, the caller answers with a clear error and nothing is half-changed.
- An update never overwrites a change it did not read.
- Where the product needs it, the order of answers is protected as well: an answer must name the index of the question it answers; a second answer for the same index gets "already answered" and is not stored.
- Shared documents (the co-use counts, a probe list) use the same versioned update; for the co-use counts a lost race is simply skipped, since a count is only a hint.

## 4.6 Lifecycle jobs

These run as scripts, by hand or on a schedule, never inside a student's request, so no request pays for them. Each prints a short report.

| job | what it does | properties |
|---|---|---|
| **Rebuild D1** | fills the problems, sessions and bank tables from their R2 documents | running it again gives the same result and no duplicate rows |
| **Sweep idle guests** | deletes every guest with no activity (any session or problem) for longer than the guest idle time, a bounded number per run | only guests, never accounts |
| **Sweep expired photos** | deletes every photo read more than 24 hours ago, a bounded number per run | the reading and the problem stay |

User-level data jobs, triggered by the student:

| action | what happens |
|---|---|
| **Sign in from a guest browser** (first time) | every object under `users/<guest id>/` moves to the account, the guest's D1 rows change owner, the guest is linked to the account, the guest's map events join the account's map, and the account's knowledge record is rebuilt from all its finished sessions, so nothing the student learned is lost |
| **Delete my data** | every object under `users/<user id>/`, the problem documents and photos they own and all their D1 rows are removed |

## 4.7 Acceptance criteria

1. Deleting every D1 row of the problems, sessions and bank tables and running the rebuild must restore the same lookups (same problem by fingerprint, same session lists, same bank items), and a second run must change nothing.
2. Two answers sent at the same moment for the same question must store exactly one; the other must be told the question was already answered.
3. Many parallel updates of one document must never lose a change that was reported as written.
4. An id containing `/`, `.`, `..` or more than 64 characters must never become part of a storage key.
5. Showing a page without an API call must read nothing from D1 or R2.
6. After "Delete my data", nothing under the user's prefix may remain and no D1 row may name the user; other users must be untouched.
