# 17. Screens

## 17.1 Overview

ThyGap will be a single-page web app that runs in the browser. Every screen has one clear next step, little text before it, and calm handling of waiting and failure. All words on screen come from one bilingual copy table (English and Korean); the student switches language at any time from the header.

| screen | address | purpose |
|---|---|---|
| Home | `/` | start: photograph or type a problem, resume an open diagnosis, open the map |
| Reading check | `/upload/{id}` | wait for the reading, then "Is this the problem?" with edit and retake |
| Splitting | `/problem/{id}` | wait for the split, then show the steps (no answers) and start the diagnosis |
| Session | `/session/{id}` | the diagnosis, one question at a time |
| Result | `/session/{id}/result` | which concept was missing, the path, the concepts touched, share, sign-in |
| Final retry | `/session/{id}/retry` | the original problem again; pass, or see the full solution |
| Map | `/map` | the star map, "Check my map", demo clock, sign-in, delete my data |
| Map check | `/map/check/{id}` | the small map with the asked star marked, one question at a time |
| Account box | on the result and map screens | Google sign-in or the signed-in email with sign out |

**The main flow:** Home → Reading check → (Splitting, only for a problem not split before) → Session → Result → Final retry or Map → Map check → Map.

**Layout.** A header with a small home link (the wordmark) on every screen except home, and the language switch. The first language follows a saved choice, else the browser's language; switching saves the choice.

### Common states
Every screen that loads data shows one of these, never a blank page:

| state | what the student sees |
|---|---|
| Loading | a skeleton in the shape of what will come |
| Working (AI in the background) | the staged progress (17.3) with its cancel |
| Error | an error card with the error's sentence in the student's language and one action, usually "Try again" |
| Not on this device | a session or check that does not belong to this browser shows one calm message and a single button to start a new problem; it is not shown as an error |
| Empty | the map with no stars says how to light it up |

## 17.2 Screens

### Home
**Purpose.** Get the problem in with as little effort as possible.

**Shows.** The brand hero (the "Thy□Gap" wordmark, the tagline "Know thy math." and a one-line subline), the heading "Stuck on a problem?", the photo picker (its camera button is the one primary button), and two quiet buttons: "type the problem instead" and "see your knowledge map". A privacy note says photos are deleted 24 hours after they are read and asks the student not to include names, faces or school names. When a diagnosis is still open in this browser, a quiet "resume" button appears.

**Next step.** A photo goes through the crop editor and is uploaded; typed text is uploaded; both open the reading check. A daily limit shows its message with "see your knowledge map" as the way on; other errors offer "try again".

### Reading check
**Purpose.** Make sure the app read the right problem before any more AI is spent.

**States.** Reading (staged progress with cancel); read (the check); failed (the reason, "try again" and "go to the start"); cancelled.

**The check.** "Is this the problem?" with the photo, the statement with its math rendered, "edit the text" and "yes, this is it". When the reading is unsure, the screen lists the hard-to-read parts, gives short photo tips and offers a retake, and editing becomes the primary action. Edits are kept in the browser until they are sent. When an edit changes a given number, the screen says which numbers no longer match and reopens the edit boxes.

**Next step.** Confirm: a problem split before goes straight into a new session; a new problem goes to Splitting. Retake asks before discarding edits.

### Splitting
**Purpose.** Show that the app understood the problem as steps, and let the student start.

**States.** Working (staged progress with cancel); ready (the split); failed (the reason, "try again" and "go to the start"); cancelled (back home).

**The split.** A "verified" badge when the split is verified (chapter 7), otherwise a quiet "draft" badge, then one row per step with its goal and the concepts it needs. A step never shows its answer.

**Next step.** "Start the diagnosis" starts a session.

### Session
**Purpose.** The diagnosis: one question at a time, with the problem at hand.

**Shows.** The problem card (with its figure) above the question, which the student can hide and show; the progress ladder; the quiz card; a quiet "start over".

**States.** Loading; not on this device; load error; active (the current question); completed (go to Result); abandoned (go home).

**Behaviour.**
- Before a probe the intro says "let us check one thing first"; before a goal "now try this step again", with the teach card or a note that the explanation is not available.
- After a wrong answer a short note says the next question looks for what is missing.
- A session keeps its language to the end; when the student's chosen language differs, a note says so.
- "Start over" asks first, then ends the session and goes home.

**Next step.** The next question, or the Result screen when the diagnosis is over.

### Result
**Purpose.** Tell the student in one line what was missing, show how they got there, and offer the next step.

**Shows.** The result card (17.3); then "The concepts you worked on", one row per concept of the path with its current map state and a "learned just now" badge where it applies; the account box when Google sign-in is on.

**States.** Loading; not on this device; load error.

**Rules.** When the student gave a known slip's answer, the card names the slip and shows that answer. Share sends a short message naming the missing concept and the question count with the site address; it never holds a question, an answer or the slip.

**Next step.** "Try the original problem again" (primary), the map, or a new problem.

### Final retry
**Purpose.** Check that the student can now connect the steps and solve the original problem.

**Shows.** "Solve the original problem", the statement, a short note that this retry checks connecting the steps and is not a full mastery test, the answer field and send.

**Behaviour.** This screen sends at once, without the undo wait. Correct → "You solved it yourself!" and the full solution; a pass is final. Wrong → "Not yet", try again or show the full solution.

**Next step.** A new problem or the map.

### Map
**Purpose.** Show what the student knows as a star map, and offer to make it more certain.

**Shows.** "Your knowledge map" with the line "brighter means more evidence that you know it", the knowledge map view (17.4), the primary "check my map", the account box, a quiet "delete my data", and, when the demo clock is on, its offset with "+7 days" and "back to today".

**States.** Empty (a line on how to light up the map; "check my map" stays available); load error; starting a check.

**Behaviour.** Moving the demo clock reloads the map so fading shows. "Delete my data" asks first ("This cannot be undone"), then says the data was deleted and shows the empty map. After a sign-in round trip the account box says the progress was saved, or that sign-in was cancelled.

**Next step.** "Check my map" opens a map check.

### Map check
**Purpose.** Ask a few questions that make the map more certain, including composed problems that join concepts and aim at known slips.

**Shows.** The small map with the asked star marked and in view; the quiz card with "map check question n" and the figure when there is one. The map updates as the student answers.

**States.** Asking; done ("Your map is up to date." and "back to the map"); not on this device; load error.

**Next step.** The next question, or back to the map.

### Account box
**Purpose.** Let a guest keep the map across devices, without pressure.

- Guest with Google sign-in on: one line "sign in to keep your map on every device" and "continue with Google", returning to the current page.
- Signed in: "signed in as {email}" and a quiet "sign out".
- Hidden when Google sign-in is off.

## 17.3 Shared pieces

| piece | what it does |
|---|---|
| **Wordmark and brand hero** | "Thy□Gap": the empty box between the words is the gap and is never filled. Read by screen readers as "ThyGap". |
| **Language switch** | A small button showing the other language's own name (한국어 / English). |
| **Photo picker** | "Take a photo" (the back camera on a phone) and "choose from the gallery"; pasting an image works on a desktop. iPhone HEIC photos are converted to JPEG in the browser. A file too large or not a photo gets a clear message and is never sent. |
| **Crop editor** | "Frame only the problem": a frame with corner handles, rotate, and "use this photo". The exported image is upright, cropped and a JPEG. |
| **Staged progress** | The stages "reading the problem", "splitting into steps and linking concepts", "checking each step", each marked done, current or to do; a note when it takes long; one quiet cancel behind a confirm. |
| **Skeleton** | Placeholder blocks in the shape of what will come; announced as loading. |
| **Error card** | The message, an optional list of details, one primary action and an optional second one. |
| **Confirm dialog** | A modal yes/no dialog; "no" is the safe default. |
| **Answer field** | One labelled field whose placeholder and phone keyboard fit the answer format. While the student types, a hint appears when the grader could not read the draft. |
| **Math keypad** | Shown under the answer field only for expression answers: x, the operators, the digits and the decimal point. |
| **Math text** | Shows text with rendered math; it can never inject markup or scripts. |
| **Quiz card** | One question: its intro, the teach card before a goal, the figure, the question, the answer field and the send bar. **Send flow:** Send (or "I don't know") starts a short undo wait with "send now" and "undo"; the answer is then sent exactly once. A network failure keeps the answer and offers "send again". Each question's draft survives a reload. |
| **Verdict badge** | A short "Correct" or a gentle "Not quite". |
| **Progress ladder** | During the steps "question n of total" with one dot per step; during the repair walk "concept check n", since that walk has no fixed length. |
| **Send bar** | The buttons of the send flow for its state: editing, waiting, sending, showing the verdict. |
| **Teach card** | "We found the missing concept", the title, the key idea, a worked example, and a fold "why it works" with the explanation. After a slip, the card clears that slip. |
| **Figure view** | Shows a figure drawn by the app's own renderer, with its description as its accessible name in the current language. A model never makes an image. |
| **Result card** | One headline chosen by the verdict, as mapped in 9.7. Under it, a note when the correct answers were all in one form ("try it another way next") and when a usual slip was seen; the draft badge when needed; the path of concepts checked; "n questions in s seconds"; retry and share. |
| **Knowledge map view** | The star map (17.4). |

## 17.4 The knowledge map view

- Only concepts the student has met are stars. Related concepts sit close, each worked problem forms a constellation (its main concept joined to the concepts it used), stars never crowd each other, and the same map always gives the same picture.
- A star's brightness shows the evidence that the student knows it; a dim star is never invisible. Its colour follows its state: known, likely known, needs a check, not checked yet, not yet known, fading.
- The map draws prerequisite lines, "used together" lines in a different style, and stronger constellation lines.
- Labels are short, and two concepts never get the same label.
- **Full mode (map screen).** Each star can be selected; the detail shows its name, state, band and area of mathematics, the forms answered, and "the parts of this concept" with each part's state and how often the usual slip on it was seen. A legend explains the states. The same map is also offered as a plain list for screen readers and small screens.
- **Compact mode (map check).** Only the drawing, with the asked star marked and in view.

## 17.5 Copy rules

The quoted lines in this chapter are key lines of planned copy, written before the event as part of this plan. During the event the Meta model will build the full bilingual copy table from them (A.1).

- **Every visible word comes from the copy table, in English and Korean.** No screen writes a sentence itself; that includes accessible names, placeholders, dialog texts, the share message, error sentences, form names, grade band names and strand names. A key used in code must exist in both languages.
- **Keys are grouped by screen or piece**, with a short prefix per group.
- **Placeholders** are named values in braces (`{name}`, `{n}`, `{email}` …); a text has exactly the placeholders its key promises, the same in both languages.
- **Fixed texts:** the app name "ThyGap" in both languages; the tagline "Know thy math." / "너의 수학을 알라"; the language switch shows the other language's own name.
- **Numbers the product promises are part of the text** (8 MB, 24 hours, +7 days, up to a minute) and must match the product rules.
- **Tone.** Plain, short, friendly words for a school student. Korean in the polite -요 style. Both languages say the same thing. A wrong answer is "Not quite", never blame; errors say what happened and what to do next. Limits are explained kindly ("you used today's free AI checks; come back tomorrow, your map is saved"). No jargon: "concept check", not "probe"; "the parts of this concept", not "elements"; "usual slip", not "pitfall" or "trap".
- **Each error code has one sentence**, and each distinct situation has a distinct sentence. Unknown codes show "something went wrong".
- **Forms, bands and strands** each have a distinct name within their group in each language (e.g. "find the value", "count the true statements", "find the value that breaks the claim", "the same idea with a twist").

## 17.6 Test-id rules

- Every element a test looks for carries one fixed test id from one shared table; tests find elements by test id or by copy key, never by a written-out sentence or by style.
- A test id is the element's **role** in kebab case (lowercase words joined by hyphens): what it is for, not how it looks or where it sits. Screen-specific ids start with the screen's short name where the role alone would be ambiguous.
- Ids are unique across the app. Repeated items (a step row, a map star) share one id and carry their identity in data attributes.

## 17.7 Accessibility and phone behaviour

- Every screen has one top heading, which takes focus on navigation.
- Every control has an accessible name from the copy table.
- Status changes are announced politely; errors are alerts.
- Everything works with the keyboard alone.
- Phone first: no sideways scrolling, drawings shrink to fit, every control is an easy tap target, and the right phone keyboard opens for each answer format.
- Reduced motion turns animation off.
- A reload never loses work: an answer draft, unsent edits and the open session come back.

## 17.8 Modules

| module | responsibility | depends on |
|---|---|---|
| Copy and language | Holds the bilingual copy table, fills placeholders, and chooses and remembers the language. | — |
| Test ids | Holds the shared table of test ids. | — |
| Shared pieces | The pieces of 17.3. | copy, test ids, browser client, grader, figure renderer, map layout |
| Screens | One per address of 17.1, plus the layout. | shared pieces, browser client, copy |

## 17.9 Acceptance criteria

**Home.** A photo or typed problem must reach the reading check in the current language; a blank text box must never be sent; a daily limit must point the student to the map; an open session must be resumable.

**Reading check.** An unsure reading must show what was hard to read and offer a retake; confirming a problem split before must start a session at once, and a new one must open the splitting screen; edits must survive a reload; a changed given number must be named and the edit boxes must reopen.

**Splitting.** The badge must show whether the split is verified or a draft; no step answer may ever appear; a failed or cancelled split must lead back to a working state.

**Session.** The problem must stay at hand above the question; a wrong answer must be followed by the note that the app looks for what is missing; the last answer must open the result; a session from another browser must show only the calm message.

**Quiz card.** An answer must be sent exactly once, after the undo wait or on "send now"; undo must keep the text; a network failure must keep the answer for "send again"; a draft must survive a reload.

**Result and final retry.** The headline must name what was missing; a seen slip must be named with the student's answer but never shared; a pass must show the full solution and be final; after a wrong retry the student must be able to see the full solution.

**Map and map check.** An empty map must say how to light it up; stars must show met concepts with their states and their parts; the demo clock must make fading visible; delete must ask first and leave an empty map; the map check must mark the asked star and end with "Your map is up to date."

**Account box.** A guest must be invited to sign in only when Google sign-in is on, and must come back to the same page.

**Everywhere.** Every visible text must exist in both languages and follow a language switch; every screen must have exactly one primary button.
