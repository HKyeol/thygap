# 5. Limits and errors

## 5.1 Product limits

These numbers are product decisions. Values not given here (timeouts, retries, waits, weights) will be chosen during the build and kept inside their module.

**Input**

| limit | value | what the student experiences |
|---|---|---|
| Largest photo | 8 MB | checked in the browser before upload (and again on the server); a bigger photo gets a clear message and is never sent |
| Accepted photo types | JPEG, PNG, WebP; iPhone HEIC is converted to JPEG in the browser | any other file gets "not a photo we can read" |
| Longest typed problem | 4,000 characters | the field stops there |
| Longest answer | 200 characters | the field stops there |

**Diagnosis**

| limit | value | meaning |
|---|---|---|
| Solution steps | at most six steps | the student sees a short, readable plan |
| Probes in one repair walk | at most 6 | the walk stays short; when spent, the session ends with what it found |
| Retries of the failed step in one repair walk | at most 4 | |
| Answers in one session | at most 25 | a session always ends |
| Known | correct answers in a row, in **at least two different forms**, at least 2 answers | one lucky answer, or two answers in one form, leave a concept *likely* |
| Confirm question | always asks another form than the first correct probe, when one exists | |

**Map and map check**

| limit | value | meaning |
|---|---|---|
| Questions in one map check | at most 8 | "Your map is up to date." after 8, or earlier when nothing is left to ask |
| Compose attempts in one map check | at most 3 | further composed questions come from the shared bank, or the target is answered with a probe |
| Concepts in one composed problem | 1 main + 0–2 supports, at most 3 in all | |
| Elements a composed problem aims at | 1 or 2, the first on the main concept | |
| Trap answers on one probe or composed problem | at most 2 | |
| Fix requests after a composed problem's read-back misses its plan | 1 | then the problem is dropped |
| Demo clock | at most 365 days ahead in total | only when demo time travel is switched on |
| Grade band the app aims at | high school year 1 (H1) | the map check may ask about every H1 concept, every concept the student has met, and all their prerequisites; the map screen shows only met concepts |

**Concept data**

| rule | value |
|---|---|
| Elements per concept | 2 to 6, at least one pitfall; a skill has none |
| Concepts listed as "used together" with one concept | 0 to 4, none of them a prerequisite or descendant of it |

**Accounts and retention**

| limit | value |
|---|---|
| Guest cookie lifetime | 1 year |
| A guest with no activity is deleted after | 180 days (accounts are never swept) |

## 5.2 Daily limits

Each user, each network and the whole app have daily limits, so one student or one attacker cannot use up the app's AI budget. Days are UTC days. Every limit is a setting; 0 switches it off. This is the one table of daily limits; the request gate of chapter 15 applies it.

| scope | uploads per day | AI requests per day |
|---|---|---|
| A user (guest or account) | 20 | 60 |
| A network (guests only, by hashed address) | 100 | 300 |
| The whole app | — | an AI cost budget per day (default 5 USD), counted from the tokens of every model call |

What counts:
- Sending a new problem costs one upload and one AI unit. Confirming a reading that starts a new split, retrying a failed reading, and retrying a failed split each cost one AI unit. A confirm that reuses a problem costs nothing.
- The unit is taken **before** the work starts; work that fails later still used its unit.
- Signed-in accounts have no network limit.
- AI work that starts later inside a session or a map check (new probes, a lesson, a composed problem) asks the same gate. When the gate says no, the app **falls back quietly**: it uses stored or built-in questions, shows the failed step without a lesson ("no lesson is available for this one"), or serves a bank problem or a probe instead of composing. The student is never stopped in the middle of a diagnosis by a limit.
- When the whole-app budget is reached, every model call is refused until the next UTC day.

## 5.3 Network, app and what the student sees

| situation | what the student sees |
|---|---|
| The request never reached the server (offline, dropped) | "Check your connection." The typed answer stays in the field; "Send again" sends the same answer at once. |
| The server answered with something that is not the app's answer | the generic message with "Try again" |
| A long AI job (reading, splitting) | stages "reading → splitting → checking", a note after a while that this can take a minute, and a Cancel button |
| An AI job failed | the reason in one sentence (took too long, could not be read or checked, cancelled) and "Try again" plus "Go home" |
| A thrown error on the server | logged on the server; the student sees the generic message, never its raw text |
| A session or check that belongs to another browser | one clear message ("this was started on another device or browser") and one button to start a new problem; no error box |
| A question that was already answered (two tabs, double send) | the screen loads the current question again |

Copy tone: short, calm, never blaming. Every message exists in English and Korean and names what the student can do next.

## 5.4 Error codes

Errors returned by the server carry one code and one short message; every code maps to exactly one message on screen. The server never writes a message by hand; it uses its one message table.

**Request errors**

| code | meaning | typical status | what the student sees | "Try again"? |
|---|---|---|---|---|
| BAD_REQUEST | the body or a value is not valid (also: the given numbers no longer match an edited reading) | 400 | on an answer: "this answer cannot be checked", the answer stays; on a confirmed reading: which given no longer matches, and the edit boxes open | no; fix and send |
| FORBIDDEN | the thing belongs to another user | 403 | "this belongs to someone else" | no |
| NOT_FOUND | missing, or another user's session or check | 404 | the other-device view (sessions, checks, results) or "not found" | no |
| ALREADY_ANSWERED | the question was already answered | 409 | the current question is loaded again | no (automatic) |
| TOO_LARGE | the photo is over 8 MB | 413 | "this photo is too large", choose another | no; choose another photo |
| NOT_AN_IMAGE | the file is not a JPEG, PNG or WebP | 415 | "this file is not a photo we can read" | no; choose another photo |
| AI_BUDGET | today's AI budget of the app is used up | 429 | "today's AI budget is used up; try again tomorrow" | yes (in case it was the last moment of the day) |
| NOT_READY | the problem is not split yet, or the reading is not finished | 409 | "not ready yet" | yes |
| CONFLICT | the session or check is no longer active, the diagnosis must finish first, or the problem was already solved | 409 | a message that fits the state; a solved retry shows the pass | no |
| USER_DAILY_LIMIT | this browser reached its daily limit | 429 | "you used today's free AI checks; come back tomorrow, your map is saved" | no |
| NETWORK_DAILY_LIMIT | this network reached its daily limit | 429 | "this network reached today's limit" | no |
| STORE_BUSY | saving failed after several tries; nothing changed | 409 | "saving failed, nothing changed; send again" | yes (Send again) |
| INTERNAL | anything unexpected | 500 | the generic message | yes |

**Job errors** (stored on an upload or a problem while a background AI job runs)

| code | meaning | what the student sees | "Try again"? |
|---|---|---|---|
| AI_TIMEOUT | the model took too long | "this took too long" | yes: retries the reading or the split |
| AI_ERROR | the model call failed | "we could not read or check this problem" | yes |
| GATE_FAILED | the model's output failed the checks after its fix tries | the same as AI_ERROR | yes |
| AI_BUDGET | today's AI budget of the app ran out while the job ran | "today's AI budget is used up; try again tomorrow" | yes |
| CANCELLED | the student cancelled | "cancelled" | no; "Go home" |

**Client-only codes**

| code | meaning | what the student sees |
|---|---|---|
| NETWORK | the request did not reach the server | "Check your connection", with Send again or Try again; nothing typed is lost |
| BAD_RESPONSE | an OK answer that is not the app's JSON | the generic message with Try again |

**Start-up errors** (never seen by a student): live AI mode without a model key refuses to start; mock Google sign-in on a public site address refuses to start.

## 5.5 Rules for the error-to-message mapping

- One small browser client handles every server call; a call always gives either the data or an error with a code and a message, never a crash.
- Every code maps to exactly one message key in the strings table; an unknown code maps to the generic message. Every key exists in English and Korean.
- AI_ERROR and GATE_FAILED share one message; the student does not need the difference.
- "Try again" is offered only where repeating can help (table above). It repeats exactly the action that failed and nothing else.
- When the browser blocks local storage, the app still works; it only forgets local conveniences.

## 5.6 Acceptance criteria

1. A photo over 8 MB, or a file that is not a photo, must get its message in the browser and never be uploaded; the server must refuse both as well when they arrive another way.
2. An upload over a user's daily limit must be refused with USER_DAILY_LIMIT before any AI work, and accepted again the next UTC day; requests at the same moment must never pass the limit together.
3. When a session needs a lesson but the daily gate says no, the session must continue and the goal question must show that no lesson is available.
4. A network failure while sending an answer must keep the answer in the field, and "Send again" must send it once.
5. Every error code (request, job and client) must show a sentence in the student's language; an unknown code must show the generic message.
6. No error answer and no screen may ever show the raw text of a server exception.
