# 10. Probes and teaching

## 10.1 What a probe is

A **Probe** is a tiny check question on one concept: one short sentence, no figure, one exact answer that code can grade. It needs only the concept and the concept's direct prerequisites, and it never names the concept it checks. The diagnosis (chapter 9) and the map check (chapter 12) both ask probes.

| field | meaning |
|---|---|
| concept | the concept the probe asks about |
| element | the element of that concept it aims at (definition, condition, procedure, representation, interpretation or pitfall), or none |
| form | compute, judge, counterexample or variant (10.2) |
| question | the text shown to the student, in the probe's language |
| expected answer | a whole number or a fraction (10.3) |
| answer format | number or fraction |
| calculation | the arithmetic that gives the expected answer; code evaluates it |
| difficulty | 0 (the simplest case) to 1 (demanding but on level) |
| language | English or Korean |
| source | built-in template or written by the model while the app runs |
| traps | for a probe aimed at a pitfall: the answer the wrong belief gives, with its calculation (10.4) |

Probes are stored per concept and language and shared by all students.

**Modules**

| module | responsibility | depends on |
|---|---|---|
| Probe gate | Checks a candidate probe in pure code; its form, answer and trap rules are shared with the composer. | calculator, grader (ch. 8) |
| Probe templates | Gives built-in probes for a few concepts, so the app works offline. Deterministic. | calculator |
| Probe generator | Asks the model for probes of one concept, gates them, and optionally has them solved independently. | model-call path (ch. 14), Probe gate, grader |
| Teach writer | Asks the model for a lesson (plain or for one pitfall), checks its shape, and caches it. | model-call path, storage |
| Content service | Finds probes and teach cards in the cheapest place first and calls the model last, behind the daily AI gate; withholds a card that would give away the answer. | the four modules above, Probe picker (ch. 9), daily AI gate (ch. 5), storage |

## 10.2 The four forms

A concept is known only after correct answers in **at least two different forms**, so the same idea must be askable in several ways. Each probe has exactly one form, and code checks that it really has that shape.

| form | what it asks | what it must look like (checked by code) |
|---|---|---|
| compute | the value the idea gives | nothing more than the general rules |
| judge | how many of a few short statements are true | a few numbered statements that test genuinely different statements about the idea; at least one true and one false |
| counterexample | a claim that sounds right but is false over a stated range: the smallest (or largest) value that breaks it | a whole-number answer; the claim really holds for some values of the range |
| variant | the idea with one condition changed, so the result differs from the plain case | an answer that is not one of the numbers in the question |

The composer (chapter 12) uses the same form rules for composed problems, with one shared implementation.

## 10.3 Answers are short and exact

- The answer is a **whole number** or a **fraction** p/q of whole numbers. A decimal is allowed only in the prob-stat strand. No multiple choice, no yes/no.
- When the natural result is a matrix, a vector, a set, a function or an irrational number, the question asks for **one number that fixes it**: an entry, a determinant, a rank, a dimension, a count, a coefficient.
- Easy probes use small whole numbers only.
- Grading is done by code (chapter 8), never by a model.

## 10.4 Traps

A **pitfall** element of a concept states a wrong belief students often hold (for matrices: "AB = O means A = O or B = O"; "AB = BA"). Every well-known slip of a concept is its own pitfall element (chapter 6). A probe aimed at a pitfall element carries one **Trap**: the answer a student who holds that belief gives, with the calculation that produces it.

- A trap answer differs from the expected answer; two traps on one item never have the same answer.
- Each aimed pitfall has exactly one trap; no trap sits on an element that is not aimed; at most two traps per probe or composed problem.
- A trap's calculation must evaluate to its answer. Trap answers are compared by value.
- A probe not aimed at a pitfall carries no trap.

When a student's wrong answer grades correct against a trap answer, the app records a **trap hit** on that pitfall element: the map counts it (chapter 11) and the teaching clears that belief (10.8).

## 10.5 The probe gate

Code checks every candidate probe, from any source, before a student sees it. It refuses:

- an empty question, or one too long for a short check;
- a question that mentions a table, or a figure (probes never carry a figure; figures come only with composed problems);
- a difficulty outside 0…1;
- a missing calculation, one that cannot be evaluated, or one whose value does not grade correct against the expected answer;
- an answer that breaks the rule of 10.3;
- an easy probe that uses a number that is not small and whole;
- a question the concept already has;
- a probe that breaks its form's shape (10.2);
- broken traps (10.4): a probe aimed at a pitfall must carry its trap correctly; any other probe must carry none.

## 10.6 Where probes come from

A probe for a slot (concept, difficulty, language, probes already used, forms to avoid, and sometimes an element to aim at) is looked for in the cheapest place first:

1. **Stored probes** of that concept and language.
2. **Built-in templates.** A few concepts have offline templates, so the app always has some probes with no model. A template gives different compute probes in the asked language, deterministic for the same inputs, with smaller numbers when easy. They are stored once made.
3. **The model**, only for a concept of the concept map and only when the daily AI gate allows it (chapter 5). It is told the questions that already exist. New probes are gated, stored and then picked from.

No model is called when an earlier place has a fitting probe. Nothing found, or any error, gives "no probe"; the diagnosis then skips the concept (unprobeable) and the map check tries another question.

**Picking:** a probe of that concept and language, not already used, near the target difficulty. A confirm attempt is never the same probe as the first attempt.

**Preferences:** a confirm prefers a new form, then the asked element. When the stored probes only offer forms the student already answered, the search goes on to the next place.

**Writing probes with the model.** The model is told the concept with its elements, its direct prerequisites (the only other concepts a probe may use), the difficulty and the form wanted; with an aimed element, that element and, for a pitfall, its trap. Each candidate passes the probe gate and must have the form and element asked for; a failed batch gets one fix try.

**Independent solve of probes** (a setting, off by default for cost): a separate model call solves each accepted probe from its question alone; a probe whose answer does not agree is dropped.

Adding probes to the store never loses probes written at the same time.

## 10.7 Teach cards

A **TeachCard** is the short lesson the student sees when the diagnosis has found the missing concept, just before the goal is asked again.

| part | rule |
|---|---|
| title | the concept, in the student's words |
| key idea | one sentence a student could remember |
| explanation | a few short plain sentences that define every term |
| worked example | a small problem with other numbers than any problem the student saw, its steps (1 to 6) and its answer |

The tone is warm and plain: no questions to the student, never "you made a mistake", math written as math.

**Caching.** A card is written once per concept, language and (for pitfall cards) pitfall, kept in storage and served to every student after that with no model call. A lesson is written once and shared.

**Where a card comes from:**

1. A stored card for that concept and language.
2. The model, only when the daily AI gate allows it.
3. Otherwise **no card**. There is no generic fallback lesson; the screen says the lesson is unavailable and the goal is asked anyway.

An unknown concept has no card.

**Leak check.** A card is withheld when it would give away the goal's answer: when the answer, in any of the forms the grader would accept, can be read off the card.

## 10.8 The pitfall teach card

When the probe that found the missing concept was answered with a trap answer, the lesson clears that wrong belief. The pitfall card names the belief as a usual slip, shows on the worked example why it gives a wrong result, and gives the right way.

- It is looked for first at the pitfall's own key, then made by the model (when allowed).
- When neither gives a card, the plain card of the concept is used (10.7).

## 10.9 Acceptance criteria

1. A judge probe whose statements are all true, or a counterexample probe whose claim never holds in its range, must be refused by the gate; every stored probe must pass the gate.
2. A probe whose calculation does not give its expected answer, or whose answer is a decimal outside prob-stat, must never reach a student; a question about a matrix must ask for one number such as an entry or a determinant.
3. A probe aimed at a pitfall must carry exactly one trap whose answer differs from the expected answer and comes from its calculation; a probe on any other element must carry none.
4. When stored probes or a template fit, no model may be called; when the daily AI gate is closed and nothing is stored, the app must go on without a probe instead of failing.
5. A confirm probe must come in a form the student has not answered whenever any source can give one.
6. The second student who needs a teach card for the same concept and language must get the stored card with no model call; a student whose wrong answer hit a trap must see the card that clears that pitfall, or the plain card when there is none.
7. A teach card that contains the goal's answer must never be shown.
