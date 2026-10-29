# 15. Accounts and security

## 15.1 Purpose

A student should be able to use ThyGap at once, with no sign-up, and still never lose what the app learned about them. So every visitor starts as a **guest** known by a signed cookie. A guest who wants the star map on another device signs in with Google; everything the guest made then moves into the **account**, and the account's knowledge record is rebuilt so nothing is lost. A student can delete all their data with one button. Guests who never come back are removed after a long idle time.

Security is kept simple and strict: a user can never act as another user, a request that only shows a page costs nothing, AI work is limited per day, every response carries security headers, uploaded files are checked by their content, and a raw error is never shown to the student.

## 15.2 Who is who

| identity | id prefix | how it is known | what it owns |
|---|---|---|---|
| Guest | `gst-` | a signed user cookie made on the first request | its uploads, problems, sessions, map, map checks, all under its own user prefix in R2 |
| Account | `acc-` | one per Google subject (the stable Google user id); the user cookie then holds the account id | the same kinds of data, plus a profile document holding the email |
| Linked guest | `gst-` | a guest that signed in once; a guest link row in D1 points it to its account | nothing of its own after the merge; requests are served as the account |

- A guest is linked to at most one account; an account can have many linked guests (one per browser it signed in from).
- The email is kept only in the account's profile document and is shown only to that account.
- The network address is stored only as a keyed hash for daily limits; the hash cannot be read back.

## 15.3 The user cookie

- The cookie holds the user id with a keyed signature made with the server secret.
- A cookie is accepted only when its signature checks. A missing, changed or badly formed cookie is ignored and the visitor becomes a **new guest** with a fresh cookie.
- The cookie is sent only by the site itself, cannot be read by page scripts, is secure outside local development, and lives one year.
- Finding the user from the cookie reads nothing from D1 or R2.

## 15.4 Google sign-in

### Design
Sign-in uses the standard OAuth authorization-code flow with PKCE and a nonce, and stores **nothing on the server during sign-in**: what the callback needs to check travels in a short-lived, signed state cookie.

Google sign-in is **enabled** when the Google client and the public site address are configured, or when the app runs with the mock Google (below). When it is off, the account box shows nothing.

### Flow

1. **Start.** The browser opens the start route with a return address. The server makes a fresh state, nonce and PKCE verifier and redirects to Google, asking the student to choose an account so a shared computer can pick the right Google user.
2. **Google** asks the student to choose an account and returns to the callback.
3. **Callback.** The server checks the state and exchanges the code at Google's token endpoint.
4. **Checks.** The Google ID token is validated per Google's OpenID Connect guidance (issuer, audience, expiry, nonce, verified email). Every failure gives one plain message: "Google sign-in failed; go back and try again." A student who cancels at Google is sent back marked "cancelled".
5. **Account.** The Google subject finds its account, or a new `acc-` account is made on the first sign-in; the profile stores the email.
6. **Merge.** When the cookie user is a guest that is not linked yet, the guest's data is merged into the account (15.6) and the guest is linked.
7. **Finish.** The user cookie now names the account and the browser goes back to the return address, marked **merged** when the merge moved anything, else **ok**. The account box shows a short notice for "merged" and "cancelled".

### Return address
The return address is a same-site path only; anything else returns to the map.

### Mock Google
For local runs and automated tests a mock Google stands in for the real one and signs in one fixed mock student through the same flow and checks. The mock refuses to start unless the site address is a local one, so it can never run on the public site.

### Who am I, and signing out
- **Me** answers whether the user is signed in, the account's email when signed in, and whether Google sign-in is enabled.
- **Sign out** is accepted only from the site itself. It gives the browser a **new guest** cookie; the account and its data stay as they are.

## 15.5 The request gate

Every request passes one request gate before any route runs.

| concern | rule |
|---|---|
| Who | The user comes from the signed cookie; no valid cookie makes a new guest and sets its cookie. |
| Daily limits | Only actions that start AI work cost units, taken before the work starts, as set in the daily-limit table of chapter 5. A full limit is refused before any work. Every other request does not touch D1. |
| Security headers | Every response, blocked or not, carries the content security policy and the referrer policy of 2.6. |
| Errors | An unexpected error is logged on the server and shown to the student only as the generic message, never with its raw text. |
| Browsing is free | A request that only shows a page reads nothing from D1 or R2. |

## 15.6 Merging a guest into an account

When a guest signs in to an account:

- Everything the guest made (problems, uploads, sessions, map checks) becomes the account's.
- Merging keeps everything both sides learned and never overwrites the account's own documents. The account's map receives the guest's events and constellations, and the account's knowledge record is rebuilt from all its finished sessions, so the map after the merge shows exactly what the answers of both sides together prove.
- Merging is safe to repeat: a second merge of the same guest changes nothing.

## 15.7 Deleting all data

"Delete my data" is one quiet button on the map screen behind a confirm dialog that says it cannot be undone.

- It deletes everything the user owns: problems and photos, uploads, sessions, map, map checks, profile, and every D1 row of the user. Nothing of any other user is touched, nor the built-in example problem, nor shared content (the problem bank, probes, teach cards, co-use counts), which holds no personal data.
- It is accepted only from the site itself and only for a known user.
- Afterwards the browser gets a **new guest** cookie, and the screen says the data was deleted and shows the empty map.

## 15.8 Idle guests and photos

- A guest with no activity (any session or problem) for **180 days** is deleted with the same deletion as 15.7. Accounts are never swept.
- The sweep runs as a maintenance job, never inside a student's request.
- A photo is removed **24 hours after it was read** by the photo sweep (chapter 4); the reading and the problem stay. The home screen tells the student this, and asks them not to photograph names, faces or school names.

## 15.9 Photos are checked by content

- A photo is accepted only when its bytes are a JPEG, PNG or WebP; the file name and the type the browser claims are ignored. Anything else is refused as "not an image" and nothing is stored.
- A photo over **8 MB** is refused before it is stored. The browser checks size and type first, but the server never trusts that.
- A stored photo is served back only to its owner.

## 15.10 Ownership

- Another user's upload, session or map check simply does not exist for a user: the answer is "not found", and nothing reveals that the id exists.
- Another user's problem answers "forbidden". The built-in example problem is open to everyone and read-only.
- Problems are reused only for the same user; another user's problem is never reused. The one exception is the built-in example problem (owner empty, 4.2): it is reused for every user.

## 15.11 Modules

| module | responsibility | depends on |
|---|---|---|
| Server basics | Gives one result shape for every service and one error shape for every route, checks photos by their content, and signs and checks the user cookie. | error codes |
| Usage gate | Counts daily units per user, per network and for the whole app, and keeps the AI budget. | daily limits store, configuration |
| Request gate | Runs on every request: finds the user, applies the limits, adds the security headers and turns errors into safe answers. | server basics, usage gate, app context |
| Sign-in | Handles Google sign-in (live and mock), who-am-I and sign out. | server basics, account and link rows, merge, object store |
| Merge | Moves a guest's data into an account and rebuilds the account's knowledge record. | object store, index rows, map store, sessions |
| Data deletion | Deletes all data of one user on request. | object store, index rows |
| Maintenance | Rebuilds the D1 index from R2 and sweeps idle guests and expired photos. | object store, index rows, data deletion |

## 15.12 Acceptance criteria

1. A first visit must get a new guest with a signed cookie and read nothing from D1 or R2; a tampered cookie must be ignored and a new guest made.
2. Every response, including errors, must carry the content security policy and the referrer policy.
3. Actions that start AI work must be refused once the daily limits of chapter 5 are reached, before any work starts; browsing must never be limited.
4. An unexpected server error must reach the student only as the generic message.
5. A mock sign-in from a guest with data must end on the return page marked "merged" with the guest's data under the account; a sign-in that fails any token check must show the plain sign-in message.
6. After a merge the account's map must show what all its finished sessions prove, and running the merge again must change nothing.
7. "Delete my data" must remove everything of that user and nothing of anyone else, and give a new guest cookie; delete and sign out must be refused from another site.
8. Idle guests must be swept after the idle time and accounts never; photos must be removed a day after reading while the problem stays.
9. A file that is not a JPEG, PNG or WebP by its content must be refused whatever its name, with nothing stored.
10. Another user's upload or session must answer "not found", another user's problem "forbidden", and the mock Google must not start on a public site address.
