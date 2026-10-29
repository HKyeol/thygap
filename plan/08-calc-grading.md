# 8. Calculation and grading

## 8.1 Why grading is code only

ThyGap's whole diagnosis rests on one judgement repeated many times: was this answer right? A step answer, a probe answer, a goal retry and every map-check question feed the student's star map, and a concept becomes known only after correct answers in at least two different forms. If a model graded, the same answer could be judged differently on two days, a confident but wrong model could mark a slip as correct, and no test could hold the behaviour fixed.

Therefore:

- **Grading is done by code, never by a model.** A model may write a problem, its expected answer and a small formula for that answer, but the decision "correct" or "wrong" is always made by the grader described here.
- **Expected values are computed, not trusted.** Where a model gives a formula for an answer (a step's answer from the givens, a trap answer from a wrong belief), the calculator evaluates it, and the gates compare the computed value with the written answer.
- **The grader forgives how an answer is written, never what it says.** "54 cm" is 54; a different value is never right.

## 8.2 The calculator

The calculator evaluates small formulas that come from a model (half the perimeter, the side from the area) and reads student expressions such as `2(x+1)`. Because formulas come from a model, the calculator is the only thing that ever runs them.

**What it supports**

- The four operations, parentheses, powers and square root (the only function).
- Variables, in two reading modes:
  - **formula mode** (for model formulas): a variable is a given's name or an earlier step's id; there is no implicit multiplication;
  - **expression mode** (for student answers): a variable is one lower-case letter, and values written side by side multiply (`2x`, `2(x+1)`).
- The precedence rules taught in school, as a student would read the formula.
- Listing the variables a formula or expression uses, each once.
- One written form of a number: equal values always print the same, wherever the app prints or compares them.

**Properties**

- **Safe.** It can only do arithmetic. The calculator can reach nothing outside the formula and never crashes on any input.
- **Deterministic.** The same formula with the same values always gives the same result.
- **Fails clearly.** Every failure is a calculation error that says what is wrong: an unknown variable (named in the message), a result that is not a finite number (division by zero), a formula that does not parse, missing input.

## 8.3 Seeded randomness

Some checks need "random" values: sample points for comparing expressions, generated numbers, the replay of a session. These come from a seeded generator, never from a system random source:

- the same seed always gives the same sequence on every machine, and a text can be turned into a seed;
- values lie in the unit interval; different seeds give different sequences;
- it is pure: no clock, no hidden state.

So the same grading call always gives the same verdict, and a session replays the same way.

## 8.4 Answer formats

Every question carries the format its answer is graded in. When a question does not say, the format is inferred from the expected answer: one number with a fraction bar is a fraction, any other single number is a number, an expression with a variable is an expression, anything else is short text.

| format | the student writes | correct when |
|---|---|---|
| number | one number, optionally with a unit | it has the same value as the expected number |
| fraction | a fraction, a mixed number (`2 1/3`) or a decimal | its value equals the expected value |
| expression | an expression in lower-case letters | it has the same variables as the expected expression and the same value at several sample points |
| choice | the number of an option | it names the same option |
| set | several items separated by commas or semicolons | the same items with the same counts, in any order |
| short text | a word or yes / no | equal to the expected text after normalising |

## 8.5 What the grader forgives

Equal values compare equal; notation never changes a value.

**The policy:** notation is forgiven, values never. The grader forgives spacing and the usual ways of writing math, one unit word at the end of a number answer (in both languages), equivalent numbers (`1/2` for 0.5), equivalent expressions (`2(x+1)` for `2x+2`), the usual ways of writing an option number, and sets in any order.

## 8.6 What the grader never accepts

- An empty answer.
- A different value, even a close one.
- Two numbers where one is asked, or a number followed by anything other than one known unit word.
- An expression with other variables, one that differs at a sample point, or one that does not parse.
- A set with a repeated or missing item, a choice that does not name exactly one option, and short text that differs after normalising.

## 8.7 Module boundaries

| module | responsibility | depends on |
|---|---|---|
| calculator | Reads and evaluates model formulas and student expressions safely, and writes every number in one form. | nothing |
| seeded random | Gives repeatable random values from a seed, and turns a text into a seed. | nothing |
| grader | Decides correct or wrong for an answer in its format, and reads the numbers out of a sentence for the gates. | calculator, seeded random |

The gates (decomposition, probes, composed problems) use the same calculator and the same number formatting, so a value the gate accepted is exactly the value the grader compares against. Extracting numbers from a sentence is how the gates check that the givens a model lists really appear in the problem text.

## 8.8 Acceptance criteria

1. A model formula such as half the perimeter minus three, evaluated with the givens, must give the expected number; a formula naming a variable that was not given must fail with an error that names it, and no formula may reach anything outside its own values.
2. The same answer graded twice against the same expected value and format must always give the same verdict, including expression answers.
3. An answer written with extra spaces or a known unit must be graded as correct when its value is right; an answer with two numbers or with any other trailing text must be graded as wrong.
4. An expression answer must be correct exactly when it has the same variables as the expected expression and agrees with it at every sample point; two different expressions must never be accepted for each other.
5. Every usual way of writing an option number must name the same option, and set answers must be compared by items and counts regardless of order.
6. Any input to the calculator must end in a value or a calculation error, never a crash or a non-finite result shown to the student.
