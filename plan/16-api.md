# 16. API

## 16.1 Principles

- **Thin routes, logic in services.** A route reads its input (the path id, a JSON body or a form), calls exactly one service operation with the request's app context and the current user, and turns the result into a response. A route holds no product rule, no storage access and no model call. This keeps every rule testable in the service, and the routes uniform.
- **One result shape.** Every service gives either a success with a status and a body, or a failure with a status, an error code, a short message and, when useful, a list of details. A success answers the body itself as JSON. A failure answers one error object holding the code, the message and the details; details appear only when there are any.
- **Error codes** are the names of chapter 5 (`BAD_REQUEST`, `FORBIDDEN`, `NOT_FOUND`, `ALREADY_ANSWERED`, `TOO_LARGE`, `NOT_AN_IMAGE`, `AI_BUDGET`, `NOT_READY`, `CONFLICT`, `INTERNAL`, `USER_DAILY_LIMIT`, `NETWORK_DAILY_LIMIT`, `STORE_BUSY`). Messages come from one table of server messages; code never writes a message by hand. The screens never show the server's message: they map the code to their own sentence in the student's language.
- **The user** is the one the request gate found from the signed cookie (chapter 15). There is no user id in any path or body. "Any user" in the table below means any visitor with a cookie, guest or account; ownership is then checked by the service.
- **Ownership.** Uploads, sessions and map checks live under their owner's prefix, so another user's id answers `NOT_FOUND`. A problem has its own document; another user's problem answers `FORBIDDEN`; the built-in example problem is open to all and read-only.
- **No answers leave the server** while they could still be asked: no question, view or next question contains an expected answer, a step's action or calculation, a checkpoint answer, a trap answer, or the final answer. The final solution is sent only by the final retry, after a pass or a give-up.
- **Background work and polling.** Reading a photo and splitting a problem run in the background after the request has answered (`202`). The screens poll the matching view until it leaves its working status. A cancel always wins over a background job that finishes later.
- **Safe writes.** The client names the question it answers; an answer to a question that is not the current one answers `ALREADY_ANSWERED` and stores nothing. A save that cannot complete answers `STORE_BUSY` and stores nothing.
- **Bodies are checked strictly** against the limits of chapter 5; anything else answers `BAD_REQUEST`.
- **Daily cost**: only the actions marked in the table take daily units, by the request gate and the daily-limit table of chapter 5. AI work inside a session or a map check falls back quietly instead of failing.

## 16.2 Routes

Paths use `{id}` for the one path segment that names the item. "Errors" lists the codes a route can give besides `INTERNAL`, `STORE_BUSY` where marked, and the daily-limit codes on the routes that cost units. Every route that costs an AI unit lists `AI_BUDGET`.

### Problems in: uploads

| method | path | takes | returns | cost | errors |
|---|---|---|---|---|---|
| POST | `/api/uploads` | either a photo as a form upload (the image file and an optional language, English or Korean) or a JSON body with the typed problem text and an optional language | `202`: the new upload id (`upl-`) and the status "reading"; the reading starts in the background | one upload unit and one AI unit | `BAD_REQUEST` (no image in the form, blank or over-long text, any other body), `TOO_LARGE` (413, over 8 MB), `NOT_AN_IMAGE`, `AI_BUDGET` |
| GET | `/api/uploads/{id}` | — | the upload view: status (reading, read, splitting, ready, failed, cancelled), the reading when there is one (statement, question, givens, answer format, choices, figure description, language, confidence, unreadable parts, and a low-confidence flag), whether there is a photo, the error code and errors, and the problem id once there is one | — | `NOT_FOUND` |
| GET | `/api/uploads/{id}/image` | — | the photo bytes with their content type, cached privately for a short time | — | `NOT_FOUND` (also for a typed upload) |
| POST | `/api/uploads/{id}/confirm` | an optional edited statement and an optional edited question (JSON) | the problem id and either "ready" (`200`, the same problem was split before and is reused with no AI) or "splitting" (`202`, a new problem is being split in the background) | one AI unit for a new split; none for a reuse | `BAD_REQUEST` (bad body; or a given number no longer in the edited text, with one detail per such given), `NOT_READY` (the reading is not finished), `AI_BUDGET`, `NOT_FOUND` |
| POST | `/api/uploads/{id}/cancel` | — | the status "cancelled" | — | `CONFLICT` (not reading or read; a split is cancelled through `POST /api/problems/{id}/cancel`, and the upload then shows "cancelled"), `NOT_FOUND` |
| POST | `/api/uploads/{id}/retry` | — | `202`, the status "reading"; the error is cleared and the reading starts again | one AI unit | `CONFLICT` (not failed in the reading; an upload that failed in the split is retried through `POST /api/problems/{id}/retry`), `AI_BUDGET`, `NOT_FOUND` |

### Problems

| method | path | takes | returns | cost | errors |
|---|---|---|---|---|---|
| GET | `/api/problems/{id}` | — | the problem view: statement, question, figure, whether it has a photo, status (splitting, checking, ready, failed, cancelled), error code and errors; once split: the step count, the two verification flags (calculation check passed, independent solve agreed) and the steps in order, each with its number, goal and required concepts named in the problem's language. Never an answer. | — | `NOT_FOUND`, `FORBIDDEN` |
| POST | `/api/problems/{id}/cancel` | — | the status "cancelled" | — | `CONFLICT` (not splitting or checking; the built-in example), `NOT_FOUND`, `FORBIDDEN` |
| POST | `/api/problems/{id}/retry` | — | `202`, the status "splitting"; the split runs again in the background | one AI unit | `CONFLICT` (not failed), `AI_BUDGET`, `NOT_FOUND`, `FORBIDDEN` |

### Diagnosis sessions

| method | path | takes | returns | cost | errors |
|---|---|---|---|---|---|
| POST | `/api/sessions` | the problem id and an optional language | `201`: the session id (`ses-`), the question index 0 and the first question (its kind: step, probe or goal; text; answer format; progress; figure; for a goal, the teach card or "unavailable") | — | `BAD_REQUEST`, `NOT_FOUND`, `FORBIDDEN`, `NOT_READY` (the problem is not split yet) |
| GET | `/api/sessions/{id}` | — | the session view: status (active, completed, abandoned), problem id, current question index and question (active only), the verdict (finished only), the retry outcome, whether the split is a draft, and the session language | — | `NOT_FOUND` |
| POST | `/api/sessions/{id}/answer` | the index of the question answered and the answer text | the verdict of this answer, the next index, and either the next question or, when the diagnosis is over, the final verdict | — | `BAD_REQUEST`, `CONFLICT` (not active), `ALREADY_ANSWERED`, `STORE_BUSY`, `NOT_FOUND` |
| POST | `/api/sessions/{id}/retry` | an empty body (open), an answer (try), or a give-up flag | open: the original statement, question, answer format and choices; try: correct or wrong, the outcome pass or fail, and on a pass the full solution in solving order; give up: the outcome fail and the full solution | — | `BAD_REQUEST`, `CONFLICT` (session not completed, or already passed), `NOT_FOUND` |
| POST | `/api/sessions/{id}/abandon` | — | the status "abandoned" | — | `CONFLICT` (not active), `NOT_FOUND` |

### Knowledge map and map check

| method | path | takes | returns | cost | errors |
|---|---|---|---|---|---|
| GET | `/api/me/map` | — | the map view: the time of the map clock; the concepts the student has met, each with its name in both languages, kind, band, strand, state, brightness, retention, streak, the forms answered, and its elements (name, state, forms, trap hits); the edges between shown concepts with their kind (prerequisite or used-together) and state; the constellations, oldest first, numbered from 1; the star layout; whether the demo clock is on and its offset in days | — | — |
| POST | `/api/demo/time-travel` | a number of whole days to move the demo clock forward (1 to 365), or a reset flag | the new offset in days | — | `NOT_FOUND` (demo clock off), `BAD_REQUEST` (also when the offset would pass 365 days ahead in total) |
| POST | `/api/map/checks` | the language | `201`: the check view: check id, status, current question index and question (with its "map check question n" progress and its figure when it has one), the number asked, and the same map view as the map screen | AI only inside, through the internal limit check | `BAD_REQUEST` |
| GET | `/api/map/checks/{id}` | — | the check view | — | `NOT_FOUND` |
| POST | `/api/map/checks/{id}/answer` | the question index and the answer text | the verdict of this answer and the new check view (the next question, or completed with no question); the map already holds the answer | — | `BAD_REQUEST`, `CONFLICT` (completed), `ALREADY_ANSWERED`, `STORE_BUSY`, `NOT_FOUND` |
| GET | `/api/concepts/graph` | — | the concept catalog version and every concept with its id, names in both languages, kind, band, strand, prerequisites and elements (id, type, names in both languages; no descriptions) | — | — |

### Account

| method | path | takes | returns | who | errors |
|---|---|---|---|---|---|
| GET | `/api/me` | — | signed in or not, the email when signed in, whether Google sign-in is enabled | any user | — |
| POST | `/api/me/delete` | an empty body | the number of problems and documents deleted; the browser gets a new guest cookie | the site itself only (origin check) | `FORBIDDEN` |
| GET | `/api/auth/google/start` | a return address (query) | a redirect to Google (or to the mock), setting the signed state cookie | any user | not found when Google is off |
| GET | `/api/auth/google/mock` | the authorization query | a redirect back to the callback with a one-time code | local mock mode only | not found outside mock mode |
| GET | `/api/auth/google/callback` | the code and state (or an error) from Google | a redirect to the return address marked merged, ok or cancelled; the user cookie holds the account | any user | a plain-text sign-in failure (400) |
| POST | `/api/auth/signout` | — | "signed out"; the browser gets a new guest cookie | the site itself only (origin check) | `FORBIDDEN` |

The four sign-in routes and "delete my data" answer redirects, plain text or cookies themselves; they are still thin: each passes the request (address, cookies, headers) to one operation of the sign-in or data deletion module.

## 16.3 The browser client

Every screen talks to the API through one small client. Requests are same-origin and carry the cookie.

- A call always gives either the data or an error with a code (chapter 5); a request that cannot reach the server gives `NETWORK`, and a success that is not the app's answer gives `BAD_RESPONSE`.
- Each code maps to one message of the screen copy (chapter 17); an unknown code maps to the generic message.
- The browser keeps a few conveniences locally (the language choice, the last open session, unsent edits and answer drafts); the app still works when local storage is blocked.

## 16.4 Modules

| module | responsibility | depends on |
|---|---|---|
| Upload routes | Photo or text in, the reading, confirm, cancel, retry, and the photo back to its owner. | uploads service |
| Problem routes | The split view, cancel and retry. | problems service |
| Session routes | The diagnosis session from start to final retry. | sessions service |
| Map routes | The map view, the demo clock, the concept catalog and the map check. | map store, map-check service |
| Account routes | Who am I, delete my data, sign-in and sign out. | sign-in, data deletion |
| Browser client | Talks to the API, maps each error code to its message, and keeps the local conveniences. | error codes, screen copy |

## 16.5 Acceptance criteria

1. Every route must read its input, call one service operation and return its result; no route may hold a product rule, a storage call or a model call.
2. Every failure answer must carry one code from chapter 5 and a message, except the sign-in redirect routes of 16.2; an invalid body must be refused.
3. A whole diagnosis, from sending a problem to the final retry, must run through these routes with the same results as calling the services directly.
4. No response, before the final retry is passed or given up, may contain an expected answer, a trap answer or the final answer of the problem being asked.
5. A full map check must run through its routes in Korean, with every question text in Korean.
6. Every error the client can meet must show a message in both languages.
