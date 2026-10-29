# 14. AI steps

## 14.1 Principles

- The app uses a model for seven jobs: reading a problem, splitting it into steps, an independent solve, probe writing, probe solving, teaching, and composing a problem from a plan. Nothing else asks a model.
- **The model writes; code decides.** Every model answer has a fixed shape. Code reads it, checks the shape, then runs that step's gate (the reading check, the solution gate, the probe gate, the composed-item gate). A failing answer goes back to the model with the error lines for another try.
- **Grading is done by code, never by a model.** Figures are data drawn by the app's own renderer, never a model image.
- Every AI step goes through **one model-call path**, so time limits, retries, cost counting and error mapping are the same everywhere.
- Every instruction text and answer shape lives in one place, so each step always sends the same instructions; every stored output is stamped with the instruction version that made it.
- In **mock mode** a stand-in model answers every step, so the whole app runs with no network and no cost.

## 14.2 Modules

| module | responsibility | depends on |
|---|---|---|
| Model instructions | Holds the instructions and the answer shape of every AI step, one set per step. | the four probe forms |
| Model call | The one path to the model: the live or mock client, cost counting, reading the answer, and the gated call loop that sends errors back for a fix. | settings, daily usage counters (ch. 5) |
| Mock model | The deterministic stand-in model of mock mode (14.6). | the worked test problem (ch. 19), the concept map |
| Config and context | Reads the settings once and gives every service what it needs (settings, storage, concept map, model call, clock, ids, background tasks). | — |

The AI steps themselves live with their chapters: Reader and Decomposer (ch. 7), Probe generator and Teach writer (ch. 10), Composer (ch. 12).

## 14.3 The AI steps

| step | purpose | inputs | outputs | rules the model must keep | code check |
|---|---|---|---|---|---|
| reading | Copy the printed problem from a photo, or clean typed text, into fields. Never solve it. | the photo or the typed text; the student's language | statement, question, givens with where they appear, answer format, figure description, unreadable parts, confidence | copy faithfully, never translate, reword or correct; never invent an unreadable part; never copy handwriting | the reading check (ch. 7) |
| splitting | Split one problem into a short chain of steps the tutor asks one at a time, tagged with concepts. | the confirmed problem and the concept catalog | final answer, main concept, support concepts, at most six steps | each step gives one value a student could type; a step question never gives away its answer; each step names the most specific concept a failing student lacks, from the catalog; the main concept is what the problem practises and never a skill; student-facing text in the student's language | the solution gate (ch. 7) |
| independent solve | Solve the whole problem from scratch and give only the final answer. | the statement and figure only; never a step | one answer | answer only | agreement with the split's final answer, graded by code |
| probe writing | Write tiny check questions on exactly one concept, each in one form. | the concept with its elements, its prerequisites, the difficulty and form wanted, sometimes one aimed element | probes with their answers, calculations and traps | short, exact and on one concept, never naming it; in the student's language; the form rules and answer rules of chapter 10; a pitfall probe carries its trap | the probe gate (ch. 10) |
| probe solving | Solve one probe independently. | the probe question only | one answer | answer only | agreement with the probe's expected answer |
| teaching | Write a short, warm lesson on one concept, or one that clears one pitfall. | the concept; sometimes the pitfall | title, key idea, explanation, worked example | the teaching rules of chapter 10; never solve the question the student will retry | shape check (ch. 10); leak check before showing |
| composing | Write one new problem from a plan. | the plan (main, supports, aimed elements, form), the language, whether a figure is needed, the numbers already used | problem, answer and calculation, figure as data or none, a worked solution, traps | the problem really needs the main, every support and every aimed element; one trap per aimed pitfall; every given appears in the statement; a figure only when asked; new numbers each time | the composed-item gate and the read-back (ch. 12) |

Each instruction's wording will be written during the build. Each answer shape is strict, with closed lists for answer formats, languages and forms.

## 14.4 The model-call path

**The live client.** The model is reached through a chat-completions style endpoint. The endpoint's JSON mode is a setting.

**Time limits and retries.** Each call has a time limit. A busy or failing server, or a timed-out try, is retried a few times; any other error fails at once.

**The gated call loop.** Every AI step uses one loop: call the model, read its output tolerantly, check the shape, run the step's gate; on errors, send them back for another try, up to the step's number of fix tries (the composer has one fix try on its gate plus its one read-back fix round, chapter 12).

**Error mapping.**

| what happened | job error code |
|---|---|
| the call timed out | AI_TIMEOUT |
| today's AI budget is spent | AI_BUDGET |
| any other failure of the call | AI_ERROR |
| the last try still failed its checks | GATE_FAILED |

These codes are stored on the upload or problem and shown with "try again" (chapter 5). Steps that run inside a session or a map check (probes, lessons, compositions) never fail the session: they give nothing and the app goes on without them.

**Cost limits.**

- The cost of every call, from its tokens and the configured prices, is added to today's whole-app counter.
- When today's cost has reached the daily budget, every further call is refused before it is sent.
- Per student and per network there are daily limits on AI work (chapter 5).
- The independent solve of probes is a setting, off by default, because it doubles the probe cost.

## 14.5 Settings

| setting | meaning |
|---|---|
| AI mode | mock (the default) or live; live only when explicitly asked |
| model endpoint, API key, model id | where the live model is; live mode without a key refuses to start |
| JSON mode | how the endpoint is asked for structured output |
| call time limit | per model call |
| independent probe check | on or off |
| daily AI budget and token prices | whole-app cost limit |
| content language | the default language of readings and content |
| mock delay | an optional wait on every mock answer, to see loading states |

## 14.6 The mock mode

A deterministic mock answers in the request's language, so the app runs end to end with no network and no cost. It reads any typed text as itself. It splits the worked test problem (chapter 19) when the text is that problem, and gives one fixed second split for any other text. Every other step answers from the worked test problem. Tests and local runs will use it. Apart from injected failures, every mock answer must pass the same gates as a live one.

**Injectable failures** (the codes of 5.4), one list for all tests:

- on any AI step: AI_TIMEOUT, AI_ERROR, AI_BUDGET, and an answer that fails its gate on every try (GATE_FAILED);
- on probes, lessons and compositions inside a session or a map check: the same failures, which the app absorbs and goes on without;
- in the browser client of the tests: NETWORK and BAD_RESPONSE.

## 14.7 Acceptance criteria

1. A model answer that cannot be read or has the wrong shape must be sent back with the error lines and never reach a student; after the last try the job must show GATE_FAILED with "try again".
2. Each kind of call failure must show its own job error; probes, lessons and compositions that fail inside a session or check must let the student continue.
3. Once today's AI budget is spent, no further model call may be sent.
4. The independent solves and the composer's read-back must never receive the steps, the plan or the expected answer.
5. With the AI mode left at its default the whole app, from upload to map check, must run with no network, in English and in Korean, and each injected failure must produce the matching error screen.
6. Live mode without an API key must refuse to start.
