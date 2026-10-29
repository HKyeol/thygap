# 9. Diagnosis

## 9.1 Purpose

A diagnosis **Session** finds the exact place where the student's knowledge stops. It has two parts:

1. **The step walk.** The app asks the steps of the problem's SolutionGraph one by one, in solving order. The first wrong answer is the **boundary**: the step the student cannot do.
2. **The repair walk.** From the failed step the app walks **down** the concept map with short check questions (**Probes**, chapter 10) until it finds the concept the student is missing. It teaches that concept with a **TeachCard**, then asks the failed step (the **goal**) again. When the goal still fails, the walk goes deeper.

The app probes before it teaches: it never teaches a concept the student has not shown to be missing, never calls a concept known on one lucky answer, and asks its confirm question in another form than the first. A wrong answer that equals a probe's **Trap** answer names the usual slip the student believes, and the teaching that follows clears that slip.

A session is stored only as the list of the student's answers (plus the probe fixed for each probe slot, so a replay asks the same). Replaying that list always gives the same questions and the same result.

## 9.2 Modules

| module | responsibility | depends on |
|---|---|---|
| Step walk | Asks the step questions in solving order and grades answers until the first wrong one. Pure. | Solution gate (solving order), grader |
| Probe picker | Chooses one probe for a concept, difficulty and language, avoiding probes already used and, for a confirm, the form already asked. Pure. | — |
| Repair walk | Walks down the concept map from the boundary. Pure. | concept map, grader |
| Session machine | Replays a session's answers through the step walk and the repair walk and says what comes next: the next question, a new probe needed, or the final verdict; it also gives the question as the student sees it. Pure: no clock, no randomness. | Step walk, Repair walk, concept map |
| Sessions service | Owns the session record: starts it, stores answers safely, fetches and fixes probes, attaches teach cards, puts a finished session onto the map, runs the final retry and abandons on request. | Session machine, Probe picker, content (probes and teach cards, chapter 10), map store (chapter 11), storage |

## 9.3 The step walk

- The questions follow the solving order of the steps. For each step the walk first asks the **extract** question when the step has one ("can you read this number from the problem?"), then always the **checkpoint** question ("can you do the step?"). These two layers are called *extract* and *reconstruct*.
- Each question carries the concepts the step requires.
- A correct answer moves on. The first wrong answer sets the boundary (the step, its layer, the concepts it requires) and ends the walk.
- All correct → the diagnosis ends with **passed**: the student could solve it step by step, and nothing is taught.
- A failure on the extract layer is an **extraction** failure (the student could not read the number); on the checkpoint it is a **reconstruction** failure.

## 9.4 The repair walk

**Targets.** The repair targets are the concepts of the failed step's requirements that can be probed: concepts of the concept map, each once, in the order the step names them. Skills and new ideas are not targets. When the step names a concept with an element (`concept#element`), the walk keeps that element as the target's element. A step with no target ends the diagnosis at once (exhausted, nothing to probe).

**The walk:**

1. It starts with a probe on the first target at a middle difficulty.
2. **A wrong probe** means the concept is missing. It becomes the *deepest* missing concept, and the next question is the goal, preceded by the teaching of that concept. When the wrong answer was a trap answer, the teaching is the card that clears that pitfall. The concept's prerequisites that are concepts (never skills) and not yet probed wait to be probed next, at an easier difficulty (the student just failed this area).
3. **A correct probe** is not proof. The walk asks a **confirm probe** on the same concept, in another form than the first. Two correct answers in a row pass the concept; then the goal is asked with nothing to teach.
4. **A correct goal** ends the walk **unlocked**; the *filled concept* is the concept taught just before it, or none.
5. **A wrong goal** sends the walk deeper: the prerequisites of the missing concept are probed before the remaining targets.
6. **Nothing left to probe** ends the walk **exhausted**, with the reason in plain words (for example, the student knows the parts but cannot yet put them together).
7. **Budget.** A walk asks at most the number of probes and goal retries set in chapter 5. These limits come before every other rule of the walk: when either is reached, the walk ends exhausted, even when prerequisites or a confirm probe are still unasked.

**Further rules:**

- Each concept is probed at most once, plus its confirm probe.
- A concept for which no probe can be found at all is recorded as *unprobeable* and skipped.
- When the walk needs a new probe, the service fetches one and fixes it for that question.
- Every answer adds one item to the walk's **path**: its kind (probe or goal), its concept, its verdict, whether it taught, and for a probe its form and element.
- **Trap hit.** A wrong probe answer that equals one of the probe's trap answers is recorded as a hit on that trap's pitfall element (the **trap hit**).

## 9.5 The session as a replayed list of answers

The session record holds the problem, its own copy of the graph, the session language, the answers in order, the **fixed probes**, the verdict, the state (active, completed, abandoned) and the final-retry outcome.

- **Replay.** On every request the session machine replays all stored answers from the start, first through the step walk, then through the repair walk. The same answers always give the same state.
- **What the machine reports:** the next question with its progress (step questions, then the repair walk), a new probe needed, or the verdict.
- **Fixed probes.** The probe chosen for a question, or the fact that none exists, is fixed for that question, so a replay asks the same even when the shared probe store grows. A session never asks the same probe twice.
- **Confirm in another form.** A confirm probe prefers another form than the first (chapter 10).
- **Answer cap.** A session takes at most the number of answers set in chapter 5, steps included; a repair walk still open then ends exhausted.
- **No answer reaches the browser.** Nothing sent out carries the expected answer, the teach concept, the pitfall, a calculation or an action.

## 9.6 Starting, answering and ending

**Start.** The problem must exist (NOT_FOUND), be the student's own or the built-in example problem (FORBIDDEN), and be *ready* with a graph (NOT_READY). The session gets a `ses-` id, its own copy of the graph, and the language asked (else the problem's); it stays in that language to the end. When the graph has a main concept, the problem's **constellation** (the main concept, then the support concepts, no repeats) goes onto the student's map at once (chapter 11).

**Answer.** Only an active session takes answers (CONFLICT otherwise). Each answer names the question index it answers; it must equal the number of answers so far, else ALREADY_ANSWERED and nothing is stored. An answer is at most 200 characters. A save that cannot complete gives STORE_BUSY and changes nothing (chapter 4). A session belongs to its student: another student's or an unknown id is NOT_FOUND.

**Teach card at the goal.** A goal question that follows a missing concept carries that concept's teach card in the session language; when the wrong probe was a trap hit, the card that clears that pitfall is asked for first. When no card can be had, or the card would give away the goal's answer, the question goes out without a card and says the lesson is unavailable; the diagnosis continues.

**End.** When the machine is done:
- the verdict is saved with the session's duration (from start to the last answer, in seconds);
- the record and its index row become *completed*;
- the session's map events go onto the map **once**, under the session id, at the time of the last answer (chapter 11), with each probe's trap hit;
- the last answer's response carries the verdict instead of a question.

**View and resume.** The view of an active session shows the current question, the answers given so far and the draft badge when the graph is a draft. A session that is not active shows its verdict only. Reloading the page mid-session resumes exactly where the student was.

**Abandon** is allowed only while active (CONFLICT otherwise); the session becomes *abandoned*.

## 9.7 The verdict

| verdict part | meaning |
|---|---|
| failure | passed, extraction or reconstruction |
| failed step | the boundary step, or none |
| outcome | unlocked or exhausted (none when passed) |
| reason | for exhausted: why the walk stopped, in words the result screen can show |
| filled concept | the concept taught before the goal was solved |
| deepest concept | the last concept found missing |
| unprobeable | concepts no probe could be found for |
| path | every probe and goal answer of the repair walk, with forms, elements and trap hits |
| counts | step questions, probes, goal retries |
| duration | seconds from start to the last answer |

The result screen (chapter 17) shows one headline per verdict:

| verdict | headline |
|---|---|
| passed | no gap found |
| unlocked, with a filled concept | the concept that was missing |
| unlocked, with no filled concept | the concepts were known and only needed recalling |
| exhausted, with a deepest concept | a concept that needs more practice (the deepest concept) |
| exhausted, with no deepest concept | stuck at a step, with the reason |

## 9.8 What a diagnosis leaves on the map

A finished diagnosis adds its evidence to the student's knowledge record (each answer's concept, kind of question, verdict, form, trap hit and time), and the star map's six states (chapter 11) are computed from it. There is one state model: the one the student sees on the map.

- Teaching and a solved goal never make a concept known. A concept becomes **known** only after correct answers in **at least two different forms** (chapter 11); correct answers in one form leave it *likely*.
- The evidence is added when the session's events are recorded (chapter 11), so recording the same session twice changes nothing.

## 9.9 Retrying the problem at the end

After a completed session the result screen offers the original problem again.

- Asking for it returns the statement, question, answer format and choices (the session must be completed, else CONFLICT).
- An answer is graded by code against the final answer and saved as *pass* or *fail*. A pass shows the step actions in solving order and is final; another answer after a pass is CONFLICT.
- A fail can be tried again, or the student can give up, which marks a pending retry *fail* and shows the full solution.

## 9.10 Acceptance criteria

1. Steps must be asked in solving order, extract before checkpoint; the first wrong answer must end the step walk, and all correct answers must end the session as passed with nothing taught and no knowledge marked unknown.
2. After a wrong step, a wrong probe on its concept must be followed by that concept's teach card and the step again; a correct step must then end the session unlocked with that concept as the filled concept.
3. A correct probe must be followed by a confirm probe on the same concept in another form; one correct probe must never make a concept known, and two correct probes in one form must leave it likely.
4. When the goal still fails, the walk must probe the prerequisites of the missing concept, within the limits of chapter 5; a skill must never be probed, and a concept must never be probed twice in one session (apart from its confirm probe).
5. A wrong probe answer that equals a trap's answer must be recorded as a hit on that pitfall element, and the teach card that follows must be the one that clears that pitfall when it exists.
6. Replaying the same answers must give the same questions and the same verdict, also after new probes were added to the shared store; reloading mid-session must show the same question.
7. No question, view or response sent to the browser may contain an expected answer, a calculation or a step action; a teach card that would reveal the goal's answer must be withheld and the screen must say the lesson is unavailable.
8. Two answers for the same question index sent at once must store exactly one; a finished session's events must reach the map exactly once.
