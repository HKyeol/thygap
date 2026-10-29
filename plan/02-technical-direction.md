# 2. Technical direction

## 2.1 Platform

| choice | why |
|---|---|
| **SvelteKit** for screens and server routes, one project | one language and one type system from the stored document to the screen; small bundles for phones |
| **Cloudflare Pages** with the Cloudflare adapter | runs close to the student, scales to zero, no server to maintain; background work after the response is kept alive by the platform |
| **D1** (SQL) as a small index | fast lookups and counts: who owns what, the same problem by fingerprint, a user's sessions, daily counters, the problem bank index |
| **R2** (object storage) for documents | every problem, session, map, check, bank item, cached lesson and probe list is one JSON document; photos are objects |
| One chat-completions model endpoint (a Meta model) | every AI step goes through one call function; the endpoint, model id and key are settings |

D1 is billed by rows read and written, so it holds only small rows that a script can rebuild from R2 (chapter 4). A document in R2 is always the truth.

## 2.2 Layers

| layer | role | may depend on |
|---|---|---|
| Contracts | the shared entity shapes (chapter 3), product constants, error codes (chapter 5), the bilingual strings and test ids | nothing |
| Engine | pure logic: calculator, grader, seeded random, concept map checks, step walk, repair walk, session machine, knowledge map, next-question choice, compose plan, map layout, figure frame, labels and drawing, probe gate, solution gate | contracts |
| Data | the concept map files (one per strand and school part) and the loader that joins them | contracts, engine checks |
| Store | D1 lookups, R2 documents with safe updates, daily counters, maintenance | contracts |
| AI | the model call, the gate loop, the mock model, and one module per AI step (reader, decomposer, probe writer, teacher, composer) | contracts, engine, store (for cost) |
| Services | uploads, problems, content, sessions, map store, map check, merge and delete, sign-in, usage gate | everything above |
| Routes | thin HTTP handlers: read the request, call one service, return its result | services |
| Screens | pages and components; talk to the server only through one client | contracts, the client |

Engine modules are pure: the same input gives the same output, the input is never changed, no clock and no network inside. Services receive one **app context** that carries the settings, the D1 and R2 bindings, the concept map, the model call, the clock, the id maker and a way to start background work. Tests will build the same context over local D1 and R2 bindings, so services are tested the way they run.

## 2.3 Strict typing

- TypeScript in strict mode everywhere; no untyped values, no `eval`.
- Every entity of chapter 3 has exactly one shape, defined once in the contracts and imported everywhere. No module invents a second shape for the same thing.
- Everything that comes from outside (request bodies, model output, stored documents read back) is checked against a schema before use. Model output is never trusted: code checks it, and on a failure sends the errors back to the model for another try.
- Formulas that come from a model (a step's calculation, a probe's calculation) run in the app's own calculator, which can only do arithmetic.

## 2.4 Deterministic behaviour

- **Same seed, same result.** Every generated value (figure numbers, sample points, mock answers, the star layout) comes from a seeded generator. The same seed gives the same sequence on every machine. Random functions of the platform are not used in the engine.
- **A session replays the same.** A diagnosis session is stored as the list of the student's answers plus, for each probe question, the probe fixed for it so a replay asks the same. Replaying the answers through the session machine always gives the same questions and the same verdict; a probe, once chosen for a question, is never replaced.
- **The map is a fold.** The star map is computed from the list of answer events and a time. Events with the same time are read in the order given. The same events and time always give the same map.
- **One clock.** All times come from the context's clock. The map has its own demo clock offset so fading can be shown on stage; map time is the server clock plus that offset.

## 2.5 Mock model mode and tests

- A setting chooses **live** or **mock** AI. Mock is the default. In mock mode a deterministic stand-in model answers every AI step from one small worked test problem, so the whole app runs with no network and no cost. It can inject the failures of 5.4 (listed in 14.6), so error screens can be shown and tested.
- **No network in tests.** Unit tests, service tests and the end-to-end runs will use the mock model, local D1 and R2, and a mock Google sign-in. No test may call a real model.
- Live mode refuses to start without a model key. Mock Google sign-in refuses to start on a public site URL.
- End-to-end runs will drive a real browser through the whole app in mock mode, at a desktop size and at a phone size.

## 2.6 Security basics

| area | rule |
|---|---|
| Identity | a user is known by a signed cookie (HMAC over the user id); a changed or unsigned cookie gives a new guest. A user can never act as another user. Finding the user reads no storage. |
| Ownership | every user document lives under that user's prefix; another user's session, upload or check answers "not found". |
| Sign-in | Google OAuth with code + PKCE and a signed state cookie; the server stores nothing during sign-in. |
| Input | every body is validated; ids in storage keys must match a strict id pattern, so user input can never reach another key; SQL values are always bound, never built from text. |
| Photos | checked by their first bytes (JPEG, PNG, WebP), never by name; size-limited before and after upload. |
| Answers | no question or view sent to the browser ever contains an expected answer, a calculation or a trap answer. |
| Output | math text is rendered without HTML injection; figures are drawn from data with every text escaped and nothing external loaded. |
| Headers | every response carries a content security policy (scripts, styles, fonts and connections only from the site; images from the site, `blob:` and `data:`; no framing; forms only to the site and to Google sign-in) and a same-origin referrer policy. |
| Errors | a thrown error is logged on the server and shown to the student as a generic message, never its raw text. |
| Privacy | the network address is stored only as a keyed hash for daily limits; the student can delete all their data with one button. |
| Cost | daily limits per user, per network and a whole-app AI budget (chapter 5) stop one student or one attacker from using up the AI budget. |

## 2.7 Phone first

- Every screen is designed for a phone held in one hand first; desktop is the wider version of the same screen.
- One primary button per screen (the next step); at most two short lines of text before it; longer text folds away.
- The answer field brings up the right keyboard; a math keypad helps type expressions; a hint explains a draft that cannot be graded before anything is sent.
- After Send there is a short undo window, then the answer goes. A draft survives a reload; a network failure keeps the answer and offers "Send again".
- The camera, gallery and paste are all ways in; iPhone HEIC photos are converted in the browser; the crop editor works with touch, mouse and keyboard.
- The look lives in one stylesheet over a fixed set of class names, so the design can change without touching any screen. The app should feel calm and easy to read.
- Accessibility: every star's state is in its accessible name and in a plain text list; dialogs keep keyboard focus inside; status changes are announced.
