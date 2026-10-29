# 12. Map check and the composer

## 12.1 Purpose

"Check my map" asks the student a short series of questions, each chosen to make the star map more certain where it matters most. Knowing the big idea of a concept is not enough: a student can still fail when one detail changes, when a usual slip is tempting, or when two concepts meet in one problem. So a map-check question is either a **Probe** (chapter 10) or a **composed problem** written on purpose from the student's map: one **main concept**, **0–2 support concepts that are really used together with it**, and **1–2 knowledge elements** of those concepts, often **pitfalls** (for matrices: AB = O does not mean A = O or B = O; AB ≠ BA). A composed problem that a student could pass knowing only the big idea is a failure of the composer.

Plans draw on the student's scope of the concept map (11.10). The map is world mathematics from elementary school to university (bands E1–U4; strands number, algebra, function, geometry, calculus, prob-stat, linear-algebra, discrete), not one country's syllabus, with every well-known slip of a concept as its own pitfall element (chapter 6). Every question is asked in one of the four forms: compute, judge, counterexample, variant.

Code, not the model, decides what to mix. The model writes the problem; code checks every part; an independent read-back that never sees the plan must find the same main concept, supports and elements, and an independent solve must agree. Passing problems go into a shared **bank** and are reused for other students.

## 12.2 Modules

| module | responsibility | depends on |
|---|---|---|
| Next question | Picks the next concept or pair of concepts to ask from the knowledge map, never one already set aside in this check. Pure and deterministic. | Knowledge map (ch. 11), concept map |
| Compose plan | Makes a **ComposePlan** for a target from the map and the co-use counts, and checks any plan against the rules of 12.4. Pure and deterministic. | Knowledge map, concept map |
| Composer | Writes one **ComposedItem** for a plan with one model call, gates it, reads it back without the plan, and gives one fix round. | model-call path (ch. 14), Solution gate and Decomposer (ch. 7), Probe gate rules (ch. 10), grader (ch. 8), figure gate (ch. 13), Compose plan |
| Map check service | Owns the **MapCheck** record: picks and serves questions (bank, composed or probe), grades answers by code, writes each answer onto the map at once. | Next question, Compose plan, Composer, Content service (ch. 10), Map store (ch. 11), daily AI gate (ch. 5), storage |
| Co-use counts | Keeps the shared co-use counts. | storage |

## 12.3 Choosing the next question

The next question is a **target**: a concept (with a reason), or a link between two concepts (reason *link* or *combo*). Only concepts are asked, never skills, and never a target already excluded in this check. The reasons are tried in this order; the first that gives a target wins:

| # | reason | what it asks |
|---|---|---|
| 1 | confirm | The last question was a concept answered correctly (not a gap question), and the concept is not known yet (one correct answer, or correct answers in one form only). Ask it again **in a form not yet answered**, because two correct answers in two forms make it known. |
| 2 | trap | A concept the map shows known or likely that has a pitfall element not yet checked, or checked and not known: the big idea seems clear, so ask the usual slip. |
| 3 | gap | Ask **the middle of the uncertain region** below a not-known concept, so one answer rules out about half. |
| 4 | fading | A known concept fading without review. |
| 5 | recheck | An old mistake, asked again at least one day later, once no concept below it is still not known (fix the bottom first). |
| 6 | half | A likely concept with one correct answer from earlier, or with correct answers in one form only. |
| 7 | combo | An untested combo edge whose two ends are both known or likely: can the student use the two together? |
| 8 | link | An untested prerequisite edge between two known concepts: does the student see how they connect? |
| 9 | explore | Ask an untested concept in the middle of unexplored ground. |
| 10 | — | Nothing left: the check ends. |

The same map always gives the same target. A target names no form and no element; the plan (12.4) or the probe search (12.8) chooses them.

## 12.4 The compose plan

A plan names: the **main** concept, the **supports** (0–2), the **targets** (1–2 pairs of concept and element) and the **form** (compute, judge, counterexample or variant).

**What makes a plan good** (the plan gate):

- Only concepts of the concept map, never a skill; main and supports all different.
- At most 2 supports; 1 or 2 targets, none twice.
- The main is not a prerequisite (direct or not) of any support.
- **Each support is really used together with the main**: it is a prerequisite (direct or not) of the main, or listed as "combines with" the main, or counted above zero in the co-use counts of the main. A concept with no such tie is refused.
- Each target is on the main or a support, with an element that concept really has; the first target is on the main.

**How a plan is made from a target** (deterministic; the same map gives the same plan):

- *Supports really used together.* The supports are concepts the student shows known or likely that are really used together with the main, preferring those most often seen with it in verified problems. For a link or combo target, the two ends become main and support.
- *First target on the main.* When the big idea of the main seems known, the first target is one of its **pitfalls** the student has not shown to know, so the problem aims at the slip; otherwise an element of the main not yet known. A support's pitfall not yet known may be the second target.
- *Forms rotate.* The form is one the student has not yet answered on the first target, so over a series of checks all four forms are asked.
- Every plan made this way passes the plan gate; a target with no possible plan has none.

Whether the problem needs a figure follows from the concepts: a plan needs one when the main or a support is marked "figure needed" or "figure helpful" (chapter 6); otherwise it carries none.

## 12.5 The composer

One model call writes the whole problem from a plan: statement, givens, question, answer with its calculation, the figure as data (or none; a figure is always data drawn by the app's own renderer, never a model image), a worked solution in the decomposer's shape, and the trap answers.

**What the model is told:** the main and each support; each target element (a pitfall target asks for its trap); the form with its rule; the language; whether a figure is needed; and which numbers this check has already used, because a check never repeats a composed problem's numbers.

**Plans refused before any call:** a main or support that is not a concept, repeated ids, more than two supports or more than three concepts, no target or more than two, a target element its concept does not have, or a first target not on the main.

**The composed-item gate** (code; every error goes back to the model as a fix line, with one fix try):

- the statement or question is not in the asked language; the statement is too long for one screen;
- a given whose number is not in the statement; an answer calculation that does not give the answer from the givens;
- a solution that fails the solution gate (chapter 7), or whose final answer is not the answer;
- **a plan miss in the solution**: its main concept is not the plan's main; a support that no step requires; a target that no step requires as that concept's element;
- the form's shape rules, the short-exact answer rule and the trap rules of chapter 10, with the aimed pitfalls as the traps that must be present;
- for a plan that needs a figure, every error of the figure check (chapter 13); for one that does not, any figure;
- numbers already used in this check (the same givens and answer).

**The read-back (round trip).** A problem that passes the gate is handed, as a fresh problem, to the same decomposer that splits student photos (chapter 7): statement, question, givens, answer format and figure. **The plan is never shown to it.** The read-back finds the plan only when all hold:

1. the decomposition succeeds and its main concept is the plan's main;
2. every support is among its support concepts or required by one of its steps;
3. every target element is required by one of its steps;
4. its independent solve agrees with the answer (chapter 7).

On a miss, the composer makes **one fix round**: each miss is sent back to the model as a fix request naming what the read-back found. The new answer passes the gate (with its own fix try) and is read back again. A second miss gives no problem.

**The kept item** records its plan, the problem and its traps, what the read-back found, the verified solution, the instruction version and the model. Its id is a `cmp-` id.

## 12.6 The bank

- A **bank key** is the main concept id plus the sorted support ids. The same key names the bank's storage folder and its index rows (chapter 4), per language.
- A new item is stored in the bank and indexed, its main and supports are added to the co-use counts, and it is served at once.
- Bank items are shared by all students; one student's composition serves the next.

## 12.7 Co-use learning

The shared **co-use counts** record, for each main concept, how often each support concept was used with it. They grow from every verified analysis of a student's problem (chapter 7) and every kept composed problem. Only concepts count, never skills. The counts are a hint, so a lost update may be skipped. The plan maker ranks supports by these counts first, so composed problems mix concepts that really appear together in problems.

## 12.8 Serving a target

For each target the service makes the plan over the student's map at the map's clock and the co-use counts, then tries sources in an order that depends on the reason:

- **trap, combo and link** targets: bank, then a new composition, then (concept targets only) a probe;
- **every other concept target**: a probe first, then bank, then a new composition.

| source | rule |
|---|---|
| bank | An item under the plan's bank key in the check's language that this check has not served and whose numbers differ from this check's other composed items; one whose form the student has not answered for the main and whose first target element is not known is preferred. |
| new composition | Only with a plan that passes the plan gate, within **at most three compose attempts per check**, and when the daily AI gate allows it. |
| probe | A probe on the concept at a middle difficulty, in the check's language, **not already used in this check** (a map check never asks the same probe twice), avoiding the forms the student already answered on that concept, aimed at the plan's first target element when there is a plan (chapter 10). |

A target that no source can serve is marked unavailable for this check and the next target is tried.

## 12.9 Answers and the map

- Every question is in the check's language. The student never sees an expected answer or a trap answer.
- Each question shows with its progress (question number of the check's total) and its figure when it has one.
- An answer names the question it answers; an answer to an earlier question gets ALREADY_ANSWERED, and one to a completed check CONFLICT. Answers are at most 200 characters. Another student's check is NOT_FOUND.
- The answer is graded by code. A **trap hit** is a wrong answer that grades correct against a trap answer; it names that pitfall element.
- The answer goes onto the map **at once** (chapter 11), under the check's id:
  - a **probe** question gives one concept event (source check) with the probe's concept, form, element and trap hit;
  - a **composed** question gives one concept event on **each plan target** (its concept and element, the plan's form, and the trap hit when the student gave that target's trap answer), then one link event from each support to the main, all with the answer's verdict.
- Then the next question is chosen from the updated map, and the response carries the new view, including the same map view the map screen gets, so the map grows while the student answers.

## 12.10 The check's limits

- A map check asks at most **8 questions**; it ends earlier when no target is left. At the end the screen says the map is up to date.
- At most **three compose attempts** are made per check; the rest come from the bank or are probes.
- A composed problem combines at most three concepts (main plus two supports) and carries at most two traps.
- The read-back gets exactly one fix round.

## 12.11 Acceptance criteria

1. Right after a first correct answer on a concept, the next question must ask the same concept in a form the student has not answered; a concept must never be called known on answers in one form.
2. After a wrong answer on a concept, the next question must be the concept in the middle of the uncertain region below it, not the concept directly below.
3. When a concept is known or likely and one of its pitfalls is unchecked, the check must ask that pitfall, through a composed problem when one can be had; a problem that mixes concepts must use only supports that are prerequisites of the main, listed as combining with it, or counted as used with it.
4. A composed problem that does not really use its plan's main concept, supports and target elements must be refused and sent back with the fault; a problem whose read-back without the plan misses the plan must get one fix round and be dropped if it misses again.
5. A kept composed problem must have been solved to the same answer by an independent solve, and must be served from the bank to the next student with the same plan without a model call.
6. A wrong answer to a composed problem that equals the trap of its first target must mark a trap hit on that pitfall, and every plan target and the support-to-main links must appear on the map at once.
7. A map check must never ask the same probe twice, never repeat a composed problem's numbers, stop after eight questions, and make no more than three compose calls; with the AI gate closed and an empty bank it must still run on probes.
8. Every question of a Korean check must be in Korean, and no view may contain an expected or trap answer.
