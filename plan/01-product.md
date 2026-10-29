# 1. Product overview

## 1.1 What ThyGap is

ThyGap helps a math student who is stuck on one problem find the exact piece of knowledge they are missing, learn it, and solve the problem. The student sends a photo of the problem (or types it). The app splits the problem into solution steps, asks the steps one by one, and when the student fails a step it walks **down** the concept map with short check questions until it finds the concept, and the element of that concept, the student does not have. It teaches that, asks the failed step again, and records everything on the student's **star map**: a picture of what the student knows, what is fading and where the doubts are. Later, a **map check** asks problems composed on purpose from that map, to test whether the student can actually use what they know.

The name is the product: *Thy Gap*. The wordmark shows an empty box between the two words, and the box stays empty. It stands for the gap.

## 1.2 Who it is for

| user | situation | what they want |
|---|---|---|
| A student up to high school year 1 (H1, the band the app aims at) | stuck on a homework or exam-prep problem, on a phone | to know *why* they are stuck, not only the answer |
| The same student over weeks | has solved several problems in the app | to see what they know, keep it bright, and find hidden weak spots |
| A guest | opens the link with no account | to start at once; sign-in is optional and only keeps the map across devices |

The app speaks English and Korean. Every visible word exists in both languages, and the student can switch at any time. A diagnosis session keeps the language it started in until it ends.

The concept map covers mathematics as it is taught around the world, from the first school year to the fourth university year (bands E1–U4), well beyond the band the app aims at. It is not one country's syllabus. A student's gap can sit far below the problem: fractions under an algebra step, row reduction under an eigenvector, the negation of a statement under an induction proof. The map has to reach that far down.

## 1.3 The problem it solves

A student who cannot solve a problem usually gets one of two kinds of help. A full worked solution shows the steps but not which step the student could not have produced alone. A "topic review" sends the student back to a whole chapter. Neither finds the real gap. And knowing a big idea is often not the gap at all: a student can know matrix multiplication and still believe that AB = O means A = O or B = O, or that AB = BA. That wrong belief is the gap.

ThyGap finds the exact gap with as few questions as possible, never teaches what the student has not shown to be missing, and names the usual slip when the student's wrong answer reveals it.

## 1.4 The core loop

| # | stage | what the student sees and does | what the app does |
|---|---|---|---|
| 1 | Photo | "Stuck on a problem?" Takes a photo, picks one from the gallery, pastes one, or types the problem. Frames the problem and turns it upright. | Checks size and type in the browser, uploads, counts the day's limit. |
| 2 | Reading check | "Is this the problem?" The photo next to the text the app read, with tips when the photo was hard to read. Can edit the text, then "Yes, this is it". | A model copies the printed problem into statement, question and given numbers; code checks the copy. If the same student split the same problem before, or it is the built-in example problem, it is reused and no AI is spent. |
| 3 | Splitting | Stages "reading → splitting → checking", then the plan of the solution: each step's goal and its concepts, never an answer. "Start the diagnosis". | A model splits the problem into at most six steps tagged with concepts; code checks every step; an independent solve must reach the same answer. |
| 4 | Step questions | One question at a time: first "can you read this number from the problem?", then "can you do this step?" | Grades by code. The first wrong answer marks the boundary of what the student can do. |
| 5 | Repair walk | "Concept check 1, 2, …": short probes on the concepts under the failed step. | Walks down the prerequisites until a probe fails; confirms passes with a second probe in another form; notes when a wrong answer equals the answer of a known slip. |
| 6 | Teach | A short lesson before the failed step is asked again: one key idea, a short explanation, a worked example. If the student's wrong answer was a known slip, the lesson clears that slip. | Serves the lesson from the shared cache or has the model write it once; refuses a lesson that gives away the answer. |
| 7 | Result | One line: which concept was missing (or that none was), how many questions and seconds it took, the concepts checked on the way, the slip if one was seen. Share, open the map, try the original problem again. | Saves the verdict and adds every answer to the star map once. |
| 8 | Star map | Every met concept is a star; brightness shows the evidence; lines show prerequisites, concepts used together and the constellation of each worked problem; slips seen on each element are listed. | Folds all answers into one state per concept and per element, with fading over time. |
| 9 | Map check | "Check my map": up to eight short questions, the asked star marked on a small map, "Your map is up to date." at the end. | Picks each question to make the map more certain; many are problems composed on purpose for this student's map (section 1.5). |

Every screen has one clear next step. Waiting, empty and error states are part of the design: a long AI job shows its stages and can be cancelled; a failed job says why and offers "Try again"; a session opened on another device gets one clear message instead of an error; an empty map invites the first problem.

## 1.5 What makes it different

1. **It finds the element, not only the concept.** Every concept on the map carries its knowledge elements: its definition, conditions, procedure, representation, interpretation, and every well-known slip as its own **pitfall** element. A check question can aim at one element. A pitfall question carries the wrong answer that the slip produces, so a wrong answer can name its cause.
2. **It probes before it teaches.** A concept is taught only after a probe on it failed. A single lucky answer never proves a concept: a pass is confirmed by a second question that asks the same idea in another form.
3. **Known means known in two forms.** A concept (or one element) is *known* only after correct answers in at least two different question forms. The four forms are: **compute** (find its value), **judge** (how many short statements are true), **counterexample** (the smallest or largest value that breaks a false claim), and **variant** (the idea with one condition changed).
4. **The map check composes problems on purpose.** Code (not a model) plans each composed problem from the student's map: one **main concept**, **0–2 support concepts that are really used together with it** (a prerequisite alone is not enough), and **1–2 knowledge elements** of those concepts, often pitfalls, in one of the four forms. A problem the student could pass knowing only the big idea is a failure. The model writes the problem; code checks it; then an independent analysis that never sees the plan reads the problem back and must find the same main concept, supports and elements (with one fix try), and an independent solve must agree. Problems that pass are kept in a shared bank for the next student.
5. **Code grades, never a model.** Every answer is compared by code. Every figure is data drawn by the app's own renderer, never an image from a model.
6. **Reproducible.** A session is stored as the list of the student's answers; the same answers always give the same questions and the same verdict.

## 1.6 Demo success criteria

The demo at the end of the event will succeed when all of these can be shown live, on a phone-sized screen, in English and in Korean:

1. A photo of a printed problem must be read, confirmed, split into checked steps and diagnosed end to end without any manual fix.
2. A diagnosis must pass through a wrong step, a wrong probe on a prerequisite, a lesson, and a correct answer to the failed step; the result must name the missing concept.
3. A wrong probe answer that equals a known slip's answer must be named on the result card and on the map, and the lesson that follows must address that slip.
4. A concept answered correctly in only one form must show as *likely*, not *known*; after a second form it must show as *known*.
5. When the demo clock moves forward a known star must visibly fade; a review must bring it back.
6. A map check must serve at least one composed problem that mixes a main concept with a support concept and aims at a pitfall; its read-back must have found the plan; the answer must light the right star and line.
7. A guest must be able to sign in with Google and keep the map; "Delete my data" must remove everything.
8. With the mock model mode, the whole loop must run with no network and no cost.
