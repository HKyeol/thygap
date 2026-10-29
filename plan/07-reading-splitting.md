# 7. Reading and splitting a problem

## 7.1 Purpose

Everything in ThyGap starts from one problem the student cannot solve. This chapter turns that problem, sent as a photo or typed text, into a **SolutionGraph**: a short chain of solution steps, each tagged with the concepts (and elements) of the concept map that a student needs for it. The diagnosis (chapter 9) asks these steps; a wrong step is where the search for the gap begins. A wrong copy of the problem or a wrong step would send the whole diagnosis the wrong way, so every model output in this chapter is checked by code before the student relies on it, and the student checks the reading with their own eyes.

The path has four stages:

1. **Read.** The model copies the printed problem into fields; code checks that the copy is faithful.
2. **Reading check.** The student sees the reading next to the photo and confirms or edits it.
3. **Split.** The model splits the problem into at most six steps tagged from the concept catalog; the solution gate checks the steps.
4. **Independent solve.** A second, separate model call solves the problem without seeing the steps; the split is *verified* only when the two agree.

Reading and splitting run in the background; the screens poll the state and show the stage.

## 7.2 Modules

| module | responsibility | depends on |
|---|---|---|
| Reader | Sends the photo or text to the model and returns a reading whose copy of the problem has been checked by code against the rules of 7.4. | model-call path (ch. 14), grader number reading (ch. 8) |
| Fingerprint | Gives one key per problem text that ignores the numbers, so the same problem sent again is found. | — |
| Decomposer | Asks the model for the steps, has the solution gate check them, sends errors back for a fix, then runs the independent solve, reporting each phase as it goes. | model-call path, Solution gate, concept map (catalog), grader |
| Solution gate | Checks a solution graph against the problem and the concept map in pure code, puts the steps in solving order and works out each step's value; it never changes its input. | calculator, grader (ch. 8), concept map (ch. 6) |
| Uploads service | Owns the upload record: stores the photo or text, runs the reader in the background, serves the reading for the reading check, and turns a confirmed reading into a problem or reuses an earlier one; handles cancel and retry. | Reader, Fingerprint, storage (ch. 4), daily AI budget (ch. 5), Problems service |
| Problems service | Owns the problem document: runs the split in the background, records each phase, serves the step preview, lets the student cancel or retry. | Decomposer, storage, co-use counts (ch. 12) |

## 7.3 States the screens see

**Upload** (the upload record, `upl-` id):

| state | meaning | next |
|---|---|---|
| reading | the model is reading the photo or text | read, failed, cancelled |
| read | a checked reading waits for the student | splitting (new problem), ready (reused problem), cancelled |
| splitting | the confirmed problem is being split and checked | ready, failed, cancelled |
| ready | the problem has a stored solution graph; a session can start | — |
| failed | reading or splitting failed; carries a job error code and the error lines | reading (upload retry, after a failed reading), splitting (problem retry, 7.9, after a failed split) |
| cancelled | the student stopped it during reading or at the reading check, or cancelled the split through the problem (7.9), which the upload mirrors | — |

**Problem** (the problem document; a problem made from an upload keeps the upload's id):

| state | meaning |
|---|---|
| splitting | the model is writing the steps and code is checking them |
| checking | the independent solve is running |
| ready | the graph is stored (verified or draft) |
| failed | splitting failed; carries a job error code and the error lines |
| cancelled | the student stopped the split (job error code CANCELLED) |

Job error codes on these records: AI_TIMEOUT, AI_ERROR, AI_BUDGET, GATE_FAILED, CANCELLED (chapter 5). The reading-check screen polls the upload; the splitting screen polls the problem and shows "splitting…" then "checking…", then the step preview.

## 7.4 Input and the reader

**Input.** A photo (cropped and rotated on the phone first, chapter 17) or typed text, in English or Korean.

- A photo may be at most **8 MB**. Its type is decided from its bytes, never from its name or the browser's claim; bytes that are not a known image are refused (NOT_AN_IMAGE); a larger photo is refused (TOO_LARGE).
- Typed text is trimmed and must be **1 to 4000 characters** (BAD_REQUEST otherwise).
- On any refusal nothing is stored. On success the record is stored as *reading*, the reader starts in the background and the answer comes at once ("accepted, reading").
- The upload remembers its language: the one the student chose, else the app's content language.
- Daily limits (chapter 5) are taken by the request gate before the upload is created.

**What the reader returns** (the reading):

| field | rule |
|---|---|
| statement | the whole printed problem, word for word, never translated, reworded or corrected; math written as LaTeX |
| question | the asking sentence, copied from the end of the statement |
| givens | one per printed number: a short lowercase name, the value, the printed unit or none, and the source text copied from the statement |
| answer format | the kind of answer asked (number, fraction, expression, choice, set, short text); options go into the choices |
| figure | whether a drawing is printed, and then a one- or two-sentence description |
| language | the printed language |
| unreadable parts | parts it could not read; never invented, marked in the text |
| handwriting | flagged when student writing is on the page; never copied |
| confidence | 0…1 |

**The copy check** (code, after every reading). Math written different ways compares equal; values never do. For example, a LaTeX fraction and the same fraction written with a slash are the same text, while a changed number is not. The check refuses, with one error line each (an error about a given names it):

- a question that is not part of the statement;
- a given whose source text is not in the statement;
- a given whose value is not in its source text;
- a figure flagged with no description.

A refused reading goes back to the model with the error lines for another try (chapter 14). A reading that still fails marks the upload *failed* with GATE_FAILED and the errors.

**Low confidence.** When the reader is not sure (low confidence, or any unreadable part) the reading-check view says so, and the screen shows tips for a better photo.

A result that arrives after the student cancelled is dropped.

## 7.5 The reading check with the student

The screen shows the photo, the reading and an edit box: "Is this the problem?" The student either confirms ("Yes, this is it") or edits the statement and the question first, then confirms.

**Confirm rules:**

- Only the statement (1–4000 characters) and the question (1–400 characters) may be edited; anything else in the request is refused (BAD_REQUEST).
- The upload must be *read*; before that it is NOT_READY.
- Edits apply to a copy of the reading. Every given's value must still appear in the edited statement; formatting may differ, values may not. Otherwise the confirm is refused with one detail line per given that does not match, naming it, and the screen asks the student to fix the text.
- **Reuse.** A problem is reused when it has the same fingerprint and the same givens, is *ready*, and belongs to the same student. Then the upload is *ready* at once with that problem, no model is called and no AI unit is spent. Another student's problem is never reused. The one exception is the built-in example problem (owner empty, 4.2): it is reused for every student.
- **New problem.** When today's whole-app AI budget is spent, the confirm is refused with AI_BUDGET. Otherwise a problem document is made from the reading (its id is the upload's id, its source "upload", its state *splitting*, no graph yet), it is indexed by fingerprint, the upload becomes *splitting* and the split starts in the background.

**Fingerprint.** The same problem with other numbers, other case, other spacing, math markers or punctuation has the same fingerprint; other words give another. Reuse also needs equal givens, so a problem with new numbers is split again.

**Cancel and retry.** The student can cancel the upload while it is *reading* or *read*; anything else is CONFLICT. A split is cancelled through the problem (7.9), and the upload then shows *cancelled* as the mirrored outcome. An upload that *failed* in the reading can be retried: the error is cleared and the reader runs again. An upload that failed in the split is retried through the problem retry (7.9). Anything else is CONFLICT.

## 7.6 The decomposer

The model receives the statement, the question, the figure description, every given with its value and unit, the answer format, and the **catalog**: one line per concept of the concept map that may matter for this text (id, name, band, can-do). It answers with the final answer and its format, the main concept, the support concepts, and **at most six steps**. Each step has:

| step field | rule |
|---|---|
| id | a short step id, unique in the solution |
| goal | what the step finds, in the student's language; shown in the preview |
| action | how the step is done; never shown before the end |
| depends on | earlier steps it uses |
| requires | the concepts it needs: catalog ids, the most specific concept a student who fails this step most likely lacks; `concept#element` only for a listed element; a new idea marked as such only when no catalog line covers the step; concepts before skills |
| givens used | the givens it reads |
| calculation | how the step's value follows from given names, earlier step values and numbers; none when the value is an expression in a variable |
| extract | only for a step that newly reads a quantity from the problem: a question whose answer is that given's value. The student is only ever asked to copy a number that is printed on the page |
| checkpoint | the question the student is asked for this step, its expected answer and format; answerable from the problem alone and never containing its own answer |

**Choosing the main concept.** The main concept is what the problem practises, not a prerequisite of a support; the structure of the problem decides it, not the solving method, and it is never a skill. Support concepts are the other concepts the steps really use.

Student-facing text (goals, questions) is written in the problem's language.

## 7.7 The solution gate

Code checks every split before anyone sees it. A graph with any error is not used; the errors go back to the model as fix lines (each error about a step names the step). The decomposer makes the same number of tries as the reader; a split that still fails marks the problem *failed* with GATE_FAILED.

**Refused:**

- *Steps.* None, or more than six; a repeated step id; a dependency on a step that does not exist; a cycle. An empty goal, action, checkpoint question or checkpoint answer.
- *Concepts.* A step that requires nothing; a reference whose concept is not in the concept map; a step whose references are all new ideas.
- *Main concept.* Empty, unknown or a skill; not what the steps practise; a prerequisite of a support concept. A support concept no step uses, or one equal to the main concept.
- *Numbers.* A calculation that cannot be evaluated; a number or fraction step without a calculation, or whose calculated value does not grade correct against its checkpoint; a final answer that differs from the last computed value (for an expression answer: one that does not grade correct against the last checkpoint).
- *Extract.* A given name that is not a given or not among the step's givens; an expected value that differs from the given's value.
- *Leak.* A checkpoint question that contains its own answer.

**Noted, not refused:** new ideas and unknown elements are noted for later growth of the concept map (chapter 6).

**Properties:**

- A calculation may use the number givens and the value of each earlier step.
- Steps are asked in a valid solving order.
- "Arithmetic checked" is true only when there are no errors and the final answer was checked.
- The gate never changes its input.

## 7.8 The independent solve

A separate model call (the **verifier**) gets only the statement and the figure description, never a step or a checkpoint, and returns one final answer. It **agrees** when its answer grades correct against the split's final answer.

A disagreement between the two solves is retried once. A solve that gives no answer leaves the split not agreed.

A graph is **verified** only when its arithmetic was checked and an independent solve agreed; otherwise it is stored as a **draft**. A draft is still usable: the session screen shows a quiet "draft" badge.

## 7.9 Background processing, preview, cancel and retry

- The split runs in the background after confirm (or retry). Each phase it reports (splitting, checking) is written to the problem's state at once, so the polling screen shows the stage.
- Success stores the graph and marks the problem *ready*; failure marks it *failed* with its code and errors. The upload, when there is one, gets the same outcome.
- **Cancel** is allowed only while *splitting* or *checking* (else CONFLICT). It marks the problem *cancelled* with no errors, and the upload, when there is one, shows *cancelled* as the mirrored outcome. A cancel always wins: after it, no later phase or result changes the problem or the upload.
- **Retry** is allowed only from *failed* (else CONFLICT); it clears the error and starts the split again, and the upload, when there is one, becomes *splitting* again. This is also how an upload that failed in the split is retried.
- **Step preview.** Once there is a graph, the problem view shows the step count, the verification flags, and the steps in order, numbered from 1, each with its goal and the names of its required concepts in the problem's language (an unknown concept shows its id). The view never contains an answer: no action, calculation, checkpoint answer or final answer.
- **Owner.** A problem belongs to its student: a missing one is NOT_FOUND, another student's FORBIDDEN. An upload is found only under its owner, so another student's upload is NOT_FOUND. The built-in example problem is open to everyone and read-only.
- **Co-use learning.** A stored graph that is *verified* adds its main concept and support concepts to the shared co-use counts once (chapter 12). A draft, a failure or a cancelled job adds nothing.

## 7.10 Acceptance criteria

1. A reading whose question is not part of its statement, or whose given does not appear where it says, must be refused and sent back to the model with one error line per fault; when the last try still fails the upload must show *failed* with "try again".
2. The same problem written with a LaTeX fraction must compare equal to its plain form; a changed number must not.
3. Editing the statement so that a given's number disappears must refuse the confirm with a line naming that given; an edit that only changes how a given is formatted must be accepted.
4. Confirming a problem the same student already split must reuse it with no model call; another student's identical problem must never be reused.
5. A split whose numbers do not hold, whose main concept is a skill, or whose question gives away its answer must be refused and either fixed by the model or marked *failed*.
6. A split must be *verified* only when its arithmetic checks and an independent solve that never saw the steps agrees; otherwise it must be a draft and the session must show the draft badge.
7. The step preview must show goals and concept names in the student's language and never an action, calculation or answer.
8. Cancelling during the split must leave the problem cancelled even when the background job finishes later; retry from *failed* must run the split again; only a verified split may add to the co-use counts.
