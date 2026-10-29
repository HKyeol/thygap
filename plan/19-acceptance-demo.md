# 19. Acceptance and demo

## 19.1 The demo script

The demo will show the whole loop in a few minutes, on a phone (mirrored) or a desktop, with one middle-school word problem. It will run on the live model; the same demo must also run end to end in mock mode, with no network, as a rehearsal and as a fallback if the network fails on stage. The demo clock will be switched on for the demo.

The demo must pass through this path:

| # | on screen | what it shows |
|---|---|---|
| 1 | **Home.** Photograph (or type) a printed word problem and frame it. | A student is stuck; one photo is enough. |
| 2 | **Reading check.** "Is this the problem?"; confirm. | The app checks its reading with the student before spending more AI. |
| 3 | **Splitting.** The steps with their goals and concepts, verified, with no answers. | Every step must have been checked by code, with an independent solve agreeing. |
| 4 | **A wrong step.** | The app does not just mark it wrong; it looks for why. |
| 5 | **A wrong probe whose answer is a known slip.** | Walking down the concept map; the wrong answer names the slip. |
| 6 | **Teaching, then the goal solved.** The teach card that clears that slip, then the step again, answered correctly. | Teaching only what was missing, then proving it on the original step. |
| 7 | **Result and final retry.** The missing concept and the slip in one line; then the original problem solved. | The exact gap, its cause, and the student now connecting the steps. |
| 8 | **Star map.** The problem's concepts as stars in one constellation; the missing concept's star brightening; its parts and the slip seen. Time travel shows stars fading. | The map is about this student, not a syllabus; knowledge fades without review. |
| 9 | **A map check with a composed problem on a pitfall.** A new problem that joins the missing concept with a concept really used together with it, aims at the slip and asks another form. | A problem built on purpose from the map and read back independently before it is asked, not a repeat of the probe. |
| 10 | (optional) **Sign in with Google.** | Guests lose nothing by signing in later. |

If time allows, a second, high-school example will show the same depth in another area: a matrix problem whose map check composes a question on the slip "AB = O means A = O or B = O".

## 19.2 Journey-level acceptance criteria

Each journey will run as a new guest in a fresh browser, on a desktop and on a phone-sized touch screen, in mock mode.

1. **Full diagnosis.** Typing the built-in example problem and confirming it must open a session at once (its split will be written with the example during the event and stored with the app, so it opens at once). The step questions must come first; after the missed step a concept check; after the wrong concept check a goal with a teach card; after the correct goal the result page, whose headline names the missing concept. The final retry with the right answer must show the pass and a solution list.
2. **New problem.** A new typed problem must show the reading stages, then the reading check; after confirming, the splitting stages, then the steps (with no answer text anywhere) and "start the diagnosis"; starting must show the first step question.
3. **Errors with retry.** A reading that fails must show an error card with "try again" and "go to the start"; a split that fails must show an error card with "try again", and trying again must succeed when the failure was temporary. A network failure while sending an answer must keep the answer, and "send again" must send it.
4. **Map.** After a finished diagnosis the map must show only the concepts of that problem as stars (including its main concept and the missing concept) and at least one constellation line; moving the demo clock forward must change its label and reload the map.
5. **Sign-in merge.** After a finished diagnosis as a guest, signing in with the (mock) Google must return to the map marked "merged" and show the account's email, and the missing concept must still be on the map (not "not checked yet").
6. **Delete data.** "Delete my data" and yes must show "your data was deleted", no stars and the empty-map message.
7. **Keyboard only.** A session must be answerable with the keyboard alone.
8. **Reload mid-session.** Reloading in the middle of a session must show the same question, with the draft typed before the reload back in the field.
9. **Phone.** On the phone-sized screen no screen may scroll sideways and every button must be an easy tap target.
10. **Map check.** A map check after a finished diagnosis must ask at most 8 questions, never the same probe twice, mark the asked star, include a composed problem when the bank or the composer can supply one, and end with "your map is up to date".
11. **Language.** Switching to Korean on any screen must change every visible text; a session started in Korean must stay Korean after switching to English, with the language note.

## 19.3 Test policy

- **Each module will have its own tests**, written together with the module, covering its rules and its acceptance criteria. Engine modules (grading, the concept graph, the walks, the knowledge map, next-question choice, layout, figures) will be tested as pure functions. Services and routes will be tested against local D1 and R2 bindings through the same app context they use in production, so they are tested the way they run.
- **No network in tests.** A deterministic mock model will answer every AI step from the worked test problem (19.4) in the request's language. The mock must be able to inject each failure of 5.4 listed in 14.6, so every error screen is testable. The mock Google sign-in will stand in for Google. Clocks and ids will be injected, so tests are deterministic: the same input and seed must always give the same result, and a session must replay the same questions.
- **Screens** will be tested with the browser client mocked: each test checks what the student sees and can do in each state (loading, working, error, empty, done) by test id and by copy key, never by CSS or a written-out sentence.
- **Contract checks** will keep the shared tables honest: every copy key must exist in both languages with the same placeholders; every key used in code must exist; test ids must be unique kebab-case role names; every class used must be a shared role and the stylesheet must style only those; colour pairs must be readable.
- **End-to-end journeys** (19.2) will run in a real browser against the running app in mock mode, on a desktop and on a phone-sized touch screen, each as a new guest.
- **Live checks** with the real model will be run by hand before the demo (one photo, one typed problem, one map check with a composed problem), never in the automated suite.

## 19.4 The small worked test problem

The tests and the mock model need one small worked example whose facts are all known; it is also the built-in example problem. Designed here; written during the event by the Meta model (A.1). It must contain:

**A small concept map** of under twenty middle-school concepts (chapter 6), a subset of the full map that follows its rules, with at least one concept that has a **pitfall** element and a few concepts **used together** with others.

**One word problem** a middle-school student can read in one breath, with given numbers traceable to the statement and a single-number answer, and a reading of it that passes every reading check.

**Its verified analysis**: a few steps that depend on each other, each tagged with its concepts, with a main concept and its supports, passing the solution gate. Its diagnosis must be able to pass through **a wrong step**, **a wrong probe whose answer is a known slip**, and **a correct goal after teaching**.

**Probes** for the concepts the walk can reach, in all four forms, including one aimed at the pitfall with its **trap**, so a confirm can ask another form and a slip can be recognised.

**Teach cards** for the concepts the walk can teach, none of which gives away an answer of the problem.

**Ways to drive a session to known outcomes** (unlocked, exhausted, confirmed in a second form), so diagnosis and map tests start from known states.

**How it is checked.** A test of the data itself will check that it keeps the rules of the plan: the small map must pass the concept-map checks, the reading and analysis must pass their gates, every probe and trap must pass the probe gate, and no teach card may leak an answer.
