# 3. Data model

This chapter names the entities the app talks about, what each one means and how they relate. The names bind the modules together: the engine, the services, the store and the screens use these entity names. Each table lists the main fields the product needs; the build will add whatever bookkeeping a module needs on top.

General conventions:
- Every time is a moment in UTC.
- Every language field is English or Korean.
- Every id that becomes part of a storage key follows the id rule of chapter 4.

## 3.1 Fixed vocabularies

| vocabulary | values and meaning |
|---|---|
| Band | the school year a concept belongs to, lowest first: elementary E1–E6, middle school M1–M3, high school H1–H3, university U1–U4 |
| Strand | number, algebra, function, geometry, calculus, prob-stat, linear-algebra, discrete |
| Concept kind | **concept** (an idea that can be probed and taught) or **skill** (a technique used inside steps; never probed, never the main concept of a problem) |
| Element type | definition, condition, procedure, representation, interpretation, **pitfall** (a wrong belief students often hold) |
| Probe form | **compute** (find its value), **judge** (how many short statements are true), **counterexample** (the smallest or largest value that breaks a false claim), **variant** (the idea with one condition changed); "the first form" means compute, then judge, counterexample and variant in that order |
| Answer format | number, fraction, expression, choice, set, short text: what the grader accepts |
| Map state | known, likely, uncertain, untested, not-known, fading |
| Question layer | extract (read a given number from the problem) or reconstruct (do the step) |

## 3.2 Learning entities

| entity | meaning | main fields | relations |
|---|---|---|---|
| **ConceptGraph** | the app's concept map: one version of all concepts with their prerequisite links | its version; its concepts | the loader joins the strand files into one; every module reads concepts through it |
| **ConceptNode** | one concept or skill | id; name in English and Korean; kind; band; strand; a one-sentence description; direct prerequisites; its elements (2 to 6 for a concept, at least one a pitfall; none for a skill); whether a problem on it needs a figure; the concepts it is often **used together with** (0 to 4, never a prerequisite or descendant of it) | prerequisites point down the map; "used together" links go both ways and form the **combo** lines |
| **Element** | one knowledge element of a concept | type; name in English and Korean; one sentence (for a pitfall, the wrong belief stated plainly) | probes, traps, composed problems and the map all point at elements |
| **Problem** | a math problem as the app understands it | statement; question; given numbers; answer format and choices; figure; language; source (upload, built-in example, or composed); owner; status | one problem may come from one upload; it has at most one solution graph |
| **Given** | one given number of a problem | name; value; unit; the text where it appears in the statement | the reading check verifies the text really appears |
| **SolutionGraph** | the checked solution of one problem as small steps | the final answer; the **main concept** (the one concept the problem is about, never a skill); the **support concepts** (the other concepts used as tools); the steps; whether the calculations hold and an independent solve agreed; the concept-map version and instruction version it was made with | belongs to one problem; a session keeps its own copy |
| **Step** | one solution step | goal in words; the steps it depends on; the concepts and elements it requires; the givens it uses; its calculation; an optional **extract question**; its **checkpoint question** | the steps are asked in a valid solving order |
| **Probe** | a short check question on one concept, optionally aimed at one element | concept; element; **form**; question; expected answer; answer format; its calculation; difficulty; language; its **traps** | shared per concept and language; once chosen for a question in a session it stays fixed there |
| **Trap** | the answer a student gives who believes one pitfall | the pitfall element; the trap answer; a calculation that produces it | a probe aimed at a pitfall carries one; a composed problem carries one per aimed pitfall (at most 2) |
| **TeachCard** | a short lesson on one concept, or one that clears one pitfall | concept and, for a pitfall lesson, the pitfall; language; title; one key idea; a short explanation; a worked example | written once per concept, pitfall and language, then shared by all students |
| **Next question** | the question the student is asked next | kind (step, probe or goal); question; answer format; progress; figure; the lesson to show before a goal | the browser never sees its expected answer |
| **PathItem** | one answered question in the repair walk | probe or goal; concept; the probe's element and form; verdict; whether a lesson was shown; the **trap hit** (the pitfall whose trap answer the student gave) | — |
| **Verdict** | the result of one diagnosis | where it failed (reading a given, doing a step, or passed); the failed step; the outcome (unlocked or exhausted); the **filled concept** (taught and then solved) and the **deepest** concept found missing; the repair path; counts and duration | drives the result screen and the map events |

## 3.3 App records

| entity | meaning | main fields | relations |
|---|---|---|---|
| **Upload** | one photo or typed text a student sent | owner; photo or text; language; the **reading** (statement, question, givens, answer format, figure, confidence, unreadable parts); status (reading, read, splitting, ready, failed, cancelled); error | becomes one problem |
| **ProblemDoc** | the stored problem document | the problem and its solution graph | one per problem |
| **Session** | one diagnosis of one problem by one student | owner; problem; its own copy of the solution graph; language (fixed for the session); the student's **answers** in order; the probes fixed for its questions; the verdict when done; status (active, completed, abandoned); start and end times | the answers are the session; everything else is replayed from them |
| **MapDoc** | one student's map document | every **MapEvent** with where it came from; the knowledge record; the demo clock offset; the **constellations** | one per user |
| **MapEvent** | one piece of evidence on the map, at a map time | a **concept** event (concept; element and form; verdict; trap hit), a **step** event (the concepts of a step question; verdict), or a **link** event (two concepts; verdict) | — |
| **Knowledge record** | per concept, the evidence the map is built from | each answer's kind, verdict, form, trap hit and time | the map's six states are computed from it |
| **KnowledgeMap** | the computed star map at one time | per concept: map state, brightness, how it fades, the **forms** of the current correct streak, and the **elements** with evidence (each with its state and trap hits); per prerequisite link and combo pair: a state and brightness | computed, never stored |
| **Constellation** | one worked problem on the map | the main concept; the concepts it used; when | joins the map's stars into a picture per problem |
| **MapView** | what the map screen receives | the met concepts with names, band, strand, state, brightness, forms and elements; the edges; the constellations; the star positions; the demo clock | computed from the MapDoc |
| **MapTarget** | what the next map-check question checks | a concept or a link between two concepts, with the reason it was chosen (chapter 12) | chosen by the next-question module |
| **ComposePlan** | what a composed problem must combine | the main concept; 0–2 support concepts; 1–2 targets (each a concept and one of its elements; the first on the main); the form | made by code from the map, never by a model |
| **ComposedItem** | a composed problem that passed every check | the problem; its traps; the plan; what the read-back found; its checked solution; instruction version; model | kept in the shared bank and reused |
| **MapCheck** | one "Check my map" run | owner; language; the questions asked and their verdicts; status (active, completed) | each answer goes onto the map at once |
| **Co-use counts** | shared counts of which support concepts verified problems use with each main concept | main concept → support concept → times seen together | grown from verified problems; one shared document |
| **Usage counter** | one day's count of one kind of work | who it counts (user, network, whole app); UTC day; kind (upload, AI call); count | the daily limits of chapter 5 |
| **Account** and **guest link** | a Google account and the guest browser that first signed in to it | Google subject; the linked guest | a guest's data moves to the account once |
| **Profile** | the signed-in email of an account | email | shown in the account menu |

## 3.4 Figures

| entity | meaning | main fields |
|---|---|---|
| **FigureSpec** | a figure as data, drawn by the app's own renderer | whether it is drawn on coordinate axes; points; segments; polygons; circles; right-angle and angle marks; equal-length ticks; labels (a number, a coordinate pair or one variable); lines; graphs of linear or quadratic functions; relations (tangent, parallel, perpendicular); the label the question asks for |

A figure is never an image from a model. The figure check accepts a figure only when it is well formed, every number label matches what is drawn and is the value of a given, the asked label's drawn size is the answer, and the figure and the problem text name the same points.

## 3.5 Model calls

| entity | meaning | main fields |
|---|---|---|
| **ModelRequest** | one call to the model | its AI step (chapter 14); the instructions; the message and images; the answer shape asked for |
| **ModelResponse** | the model's reply | the text; tokens in and out; the model id |
| **Gate outcome** | the result of a checked model call | the checked value, or a job error code (AI_TIMEOUT, AI_ERROR, AI_BUDGET, GATE_FAILED) |

## 3.6 How the entities fit

- An **Upload** becomes a **Problem**; splitting adds its **SolutionGraph**. A **Session** copies the graph, collects answers, fixes the **Probes** it asks, and ends with a **Verdict** whose **PathItems** become **MapEvents** in the student's **MapDoc**.
- The **KnowledgeMap** is computed from the MapDoc's events and the concept map at a time; the **MapView** is that map restricted to the concepts the student has met.
- A **MapCheck** picks **MapTargets** from the KnowledgeMap; for each it makes a **ComposePlan** and serves a **ComposedItem** from the shared bank or a new one, or a **Probe**. Every answer adds MapEvents at once.
- **Traps** connect a wrong answer to a **pitfall Element**; the hit is kept on the PathItem, the MapEvent and the element's trap count on the map, and selects the pitfall **TeachCard**.
- **Co-use counts** grow from every verified SolutionGraph and every kept ComposedItem, and guide which supports a plan may use.
