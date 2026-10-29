# Pre-existing work

- `plan/`: the ThyGap planning document, written with AI tools before the event. It describes the product, its rules and its tests in words, and quotes some key lines of on-screen copy (chapter 17, including the tagline in English and Korean); it contains no code, test code or data files.
- The build script (kept outside this repository), prepared before the event, sends the work cards one at a time to the Meta model with a short general instruction to write the files each card asks for, saves the files it returns, runs the checks and tests, and sends failures back. It holds no app code, tests, data or user-facing text.

README.md and this file were also written before the event.

Everything else in this repository will be made during the event by a Meta model from the plan. The build log will be kept in `build-log/`: one file per work card, with the Meta model and the time.
