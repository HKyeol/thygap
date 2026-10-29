# 20. Appendix: How the app is built during the event

## A.1 What exists before the event and what is made during it

| prepared before the event | made during the event |
|---|---|
| this planning document, README.md and PRE-EXISTING.md | every source file of the app |
| the build script (kept outside this repository) | every test |
| | the contracts: entity types, constants, error codes and messages, and the full bilingual copy table and the test ids, built from the key lines and naming rules of chapter 17 |
| | all data: the concept map files, the built-in example problem (the worked test problem of 19.4) and its test data, figure test data |
| | every model instruction and answer schema |
| | the stylesheet |

**Disclosure.** This planning document was written with AI tools before the event. The build script, prepared before the event, sends the work cards one at a time to the Meta model with a short general instruction to write the files each card asks for, saves the files it returns, runs the checks and tests, and sends failures back. It holds no app code, tests, data or user-facing text. During the event a Meta model will split the plan into work cards and write every source file, test, data file, model instruction and string. The build log will be kept in `build-log/`: one file per work card, with the Meta model and the time.

## A.2 The method

1. **Split the plan into work cards.** A Meta model will read this plan and split it into small work cards, in build order. Each card must name the files it makes, the names it may use from earlier cards, the rules from the plan it must keep, and the tests that prove it. A card is small enough to be written in one answer.
2. **Contracts first.** The first work card writes the project's package list from the platform choices of chapter 2. The next cards will generate the contracts from the plan: the entity shapes from chapter 3; the product constants and the error codes with their messages from chapter 5; the full bilingual copy table and the test ids from the key lines and naming rules of chapter 17. Every later card will meet the others only through these names, so no two cards invent two shapes for one thing.
3. **Then every module, with its tests.** In dependency order: engine modules, concept data, storage, the AI steps with the mock model, services, routes, screen components, screens, end-to-end runs, and the stylesheet last. Each card will write its code and its tests in the same answer. Each acceptance criterion in this plan must become at least one test.
4. **The build script will run every card.** For each card the script will save the files, format them, check the types, and run that card's tests. On a failure it will return the failing lines to the model, which answers with corrected files; the loop will repeat until the card passes. Then the next card starts.
5. **Whole-app proof.** After the last card, a real browser will run through the whole app in mock mode on a desktop size and a phone size.

## A.3 Rules every card keeps

1. Make exactly the files of the card, whole, with no placeholders, no skipped tests and no "to do".
2. Use only names that earlier cards made; never redefine a contract.
3. Where the card does not say a detail, choose the simplest way that keeps the plan's rules. Numbers the plan does not give are chosen during the build.
4. Install no packages beyond the project's list; call no real model; use no network in tests.
5. Strict typing everywhere.
6. Screens use only the agreed class names and the strings table; no inline styles, colours or sizes. The look lives in one stylesheet written last.
7. Tests find elements by test id and by text from the strings table, never by styling.

## A.4 Why this order is chosen

- Contracts first will let every later card be written and tested alone, against fixed names.
- Pure engine modules come before anything with storage or a model, so most of the product's rules will be proven early with fast tests.
- The mock model comes before the services that call AI, so every service, route and screen will be tested end to end with no network and no cost.
- The stylesheet comes last, so design changes will not break a test.
