# 11. The knowledge map

## 11.1 Purpose

The **KnowledgeMap** is the student's "star map": every concept the student has met, how sure the app is that the student knows it, which of its elements were checked, which usual slips the student fell into, and how concepts connect. Every answer the student gives, in a diagnosis or a map check, is a **MapEvent**; the map is folded from all events at a given time, so knowledge fades when it is not reviewed. The map screen, the next-question choice of the map check and the composer (chapter 12) all read it.

## 11.2 Modules

| module | responsibility | depends on |
|---|---|---|
| Knowledge map | Folds events into one state and brightness per concept, per element and per edge at a given time, including how much has faded; also turns a finished session into its map events. Pure and deterministic. | concept map (ch. 6), Step walk (question order) |
| Map store | Owns the student's map document: records events and the knowledge record without losing a write, records constellations, builds the map view and the read-only concept-map view, and runs the demo clock. | Knowledge map, map layout (17.4), storage |

## 11.3 Events

| event kind | made by | fields |
|---|---|---|
| concept | a probe in a diagnosis, a goal solved after teaching, a map-check question | the concept, the element asked (or none), the form (or none), the verdict, the time, where it came from (probe, goal or check), and the trap hit (the pitfall element whose trap answer the student gave, or none) |
| step | a step question of a diagnosis | the real concepts the step exercises, the verdict, the time |
| link | a map-check question on two concepts together | from-concept, to-concept, the verdict, the time |

**From a finished diagnosis:** each step question whose step exercises real concepts gives one step event with its verdict; each probe on the repair path gives a concept event (source probe) with the path item's element, form and trap hit; a goal that was taught and then answered correct gives one correct concept event (source goal, no element, form or trap hit). Nothing else gives an event. All events of a session share the time of its last answer.

**From a map check:** see chapter 12.

## 11.4 Time and order

- The map depends only on its events and the time it is asked for; the events are never changed.
- Events are read in time order, and **events with the same time are read in the order they were given**.
- An id that is not in the concept map, or a link that is not an edge, changes nothing.
- The map's clock is the real time plus the demo offset (11.11).

## 11.5 The six states

For each concept the first state that fits wins:

| state | meaning |
|---|---|
| not-known | the last answer on this concept was wrong |
| uncertain | a concept that builds on it was answered wrong and it has no correct answer of its own yet: the app doubts it |
| fading | it was known, but too long ago without review |
| known | at least **two correct answers in a row, in at least two different forms** |
| likely | one correct answer; correct answers in one form only; or enough credit from concepts that build on it |
| untested | nothing says anything yet |

Each concept also has a **brightness** from 0 to 1: more correct answers give more, wrong answers less, and it falls with retention. A wrong answer sets it to 0.

## 11.6 Credit and doubt

- **Credit flows down.** A correct answer gives credit to the concept's prerequisites, less the farther away they are, and never more than a direct answer. The direct prerequisites become *likely*; farther ones only get brighter. A correct step gives credit to the step's concepts and, less, to their prerequisites.
- **Doubt spreads down.** A wrong answer makes the concept *not-known* and casts doubt (*uncertain*) on **all** its prerequisites below it, direct and farther, that have no correct answer of their own.
- A later correct answer on the concept removes the doubts it raised.
- A prerequisite of a *known* concept is never doubted.

## 11.7 Elements and trap hits

Each concept node lists its elements that have at least one event naming them, in the concept's element order:

- an element's own **state** comes from the events that ask it, by the same streak, two-form and fading rules as a concept, without credit or doubt from other concepts;
- its own **forms**: the distinct forms of its current correct streak;
- **trap hits**: how many wrong answers on the concept hit this element's trap.

A node's **forms** are the distinct forms of the events in its current correct streak, in the fixed form order (compute, judge, counterexample, variant). An event with no form counts toward the streak but adds no form.

## 11.8 Fading and reviews

- Retention is full right after an answer and falls over time.
- A concept known but not reviewed for too long becomes *fading*, with a lower brightness.
- A **review** is a correct answer on a known concept at least one day after it was last answered. Each review makes the concept fade more slowly, so it stays bright longer.

## 11.9 Edges

| edge kind | between | from the concept map |
|---|---|---|
| prerequisite | a concept and each of its direct prerequisites (from the prerequisite to the concept) | the prerequisite lists |
| combo | two concepts often used together in problems without one building on the other | the "combines with" lists, each pair once |

An edge is *known* after a correct link question, *not-known* after a wrong one, else *untested*.

## 11.10 The map document and recording

Each student has one map document: every event with where it came from (session or check), the knowledge record (the evidence the map is built from, chapter 9), the demo clock offset, and one **constellation** per worked problem (the session id, the main concept first, then the concepts it used, no repeats).

- A student with no map document has an empty map; reading never writes.
- Two writers at the same time never lose each other's change (chapter 4).
- **Recording is idempotent.** Recording the same session, check answer or constellation twice changes nothing.

**Scope.** The part of the concept map a student's map is about: every concept of the app's target band, every concept the events name, and any extra ids asked for, each with all its prerequisites, in concept-map order. Combo lines are kept only between concepts inside the scope.

## 11.11 What the map screen shows

- Only **met** concepts: those of the constellations and the events. A new student's map is empty.
- For each: its name in both languages, band, strand, state at the map's clock, brightness, retention, streak, forms, and its checked elements with their names, states, forms and trap hits (the slips seen on that element).
- Only edges whose two ends are shown, both kinds, each with its kind.
- Constellations oldest first, labelled 1, 2, 3 ….
- A star position for every shown concept, with the concepts of one worked problem grouped around its main concept.
- Whether the demo clock is on, and its offset in whole days.

A separate read-only view gives the whole concept map (ids, names, kinds, bands, strands, prerequisites and elements without their descriptions) for the screens.

**Demo clock (time travel).** When switched on for a demo, the student can move the map's clock forward by a whole number of days, at most 365 days ahead in total, or reset it to zero, to show fading on stage. When switched off, the demo clock does not exist for the student.

## 11.12 Acceptance criteria

1. Two correct answers on a concept in two different forms must make it known; two correct answers in the same form must leave it likely.
2. A wrong answer must make the concept not-known with brightness 0 and every prerequisite below it that has no correct answer of its own uncertain; a prerequisite of a known concept must never be made uncertain.
3. A correct answer must make the direct prerequisites likely and farther prerequisites brighter, never brighter than the concept answered.
4. A known concept left without review long enough must show fading; a review a day or more later must keep it bright longer than without the review.
5. Two wrong answers that hit the same pitfall's trap must show that element with two trap hits.
6. The map screen must show only met concepts, only edges between shown concepts, and every constellation in order; recording a finished session a second time must change nothing.
