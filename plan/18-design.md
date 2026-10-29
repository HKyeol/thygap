# 18. Design principles

## 18.1 The feeling

A student who opens ThyGap is stuck and probably a little frustrated. The app should feel calm, quiet and easy to read, and always make the next step obvious. It is a patient tutor, not a game and not a test: no timers on screen, no scores in red, no crowded panels.

## 18.2 Principles

1. **One primary button per screen.** Each screen has exactly one filled, full-width button: the next step (take a photo, yes this is it, start the diagnosis, send, try the original problem again, check my map, back to the map). A screen that is waiting or failing has the error card's action as its primary step.
2. **Quiet secondary actions.** Everything else is visibly lower: a quiet (outlined) button for real alternatives (choose from the gallery, type the problem, share, sign out, delete my data, +7 days), and a plain text button for low-weight escapes (I don't know, undo, show the full solution, go to the start).
3. **Little text before the action.** At most two short lines before the primary button. Anything longer (the privacy note, the reason a teach card works, the worked example's steps) goes in a fold: a closed summary line the student can open.
4. **Calm feedback.** A verdict is a small pill ("Correct" / "Not quite") with a gentle motion that is switched off under reduced motion. A wrong answer is never loud: no shaking, no red screen. Errors appear in a card with a calm marker, say what happened and what to do, and keep the student's work.
5. **Show progress, not pressure.** Step progress and the staged progress read as one bar of equal parts (done, current, to do). The repair walk shows only "concept check n", never a total that might feel endless.
6. **Accessible by default.** Every control has an accessible name in the current language; icon buttons too. Every text colour is readable on its background (contrast at least 4.5 : 1). Focus is always visible. Everything works with the keyboard alone. Status changes are announced politely, errors as alerts. Colour is never the only signal: states also carry words (in star names, the legend and the text list).
7. **Phone first.** Designed for a phone held in one hand, then widened for a desktop. No sideways scrolling; drawings shrink to fit; buttons and fields are easy tap targets (at least 40 CSS pixels); the compact map scrolls inside its own box.
8. **Math reads like math.** Math in questions is rendered properly. Answers are typed in a monospace face so `x+3` and `3x` are easy to tell apart.
9. **Korean is a first-class language.** A system font stack that covers Korean well; layouts tolerate the different lengths of the two languages.
10. **Our own drawings.** Figures and the star map are drawn by the app as SVG from data. The design styles them through state attributes, never through pictures.

## 18.3 The look in one stylesheet

The whole look lives in **one stylesheet**. Screens and pieces never carry inline styles, style blocks, colours or sizes; they only name a small shared set of class roles and set state attributes. The design can therefore change completely by changing that one stylesheet, with no change to any screen. The colour tokens and class roles below are planned names; the build may rename them as long as the roles and the contrast rules hold.

### Colour tokens
Colours are named tokens defined once at the top of the stylesheet, at least:

| token | meaning |
|---|---|
| primary | the main action, focus rings, the current step |
| on-primary | text on any filled colour |
| accent | warm: the teach card, the missing concept, constellation lines, the wordmark's box |
| correct | the ok verdict, known stars |
| wrong | the not-quite verdict, gap rows |
| text | body text |
| muted | hints and secondary text |
| bg | the page background |

Tints and shades may be added. Every text/background pair in use (text on bg, muted on bg, on-primary on primary, primary on bg, accent on bg, on-primary on correct and on wrong, and every added tint pair) is checked to be readable.

### Class roles
A small, fixed set of class roles, grouped by purpose (the build will fix the exact names once, and no screen may invent another):

| group | roles |
|---|---|
| Layout | the page column; a vertical stack; a horizontal row; a two-column grid that falls back to one column on a phone; a card |
| Type | the hero title; the question title; a hint line; a visually hidden text for screen readers only |
| Buttons | the base button; primary; hero (the large primary on home); quiet; text; icon-only |
| Input | the field; the answer field (monospace); the keypad; a key; an operator key |
| Badges | the base pill; ok; bad; unknown (draft); step (question intro); gap (found concept) |
| Content blocks | the fold; the problem card; the teach card with its key idea and its example; the result hero; the path (trail) and a gap row in it; the error card |
| Map | the compact map box; the legend |
| Brand | the wordmark at small and large size; the empty box of "Thy□Gap" |

### State attributes
Drawn and changing parts are styled by attributes, not classes: which part of a drawing an element is (a star's dot, tap area, label, an element mark, a constellation line, a legend sample), its state (done, current, to do; the six map states; edge states), whether it is focused (the asked star), and whether motion is on. The stylesheet styles only the shared roles and state attributes.

## 18.4 Specific looks

- **Progress:** equal segments in one bar; done segments filled, the current one in the primary colour, to-do segments empty.
- **Badges:** small pills whose colour follows their meaning: ok, bad, unknown, step, gap.
- **Teach card:** the accent colour, with the key idea standing out; the example set apart.
- **Problem card, result hero, error card:** each has a calm marker in its own colour (for example a side stripe), so the student recognises them at a glance.
- **Star map:** each dot coloured by its state and more opaque when brighter; the asked star ringed; labels readable over lines; constellation lines stronger than prerequisite lines; "used together" lines dashed or otherwise distinct.
- **Wordmark:** "Thy□Gap" with the box an empty outlined square, the gap that is never filled.
- **Motion:** calm and short on the verdict badge only; none under reduced motion.

## 18.5 Acceptance criteria

1. Every screen must have exactly one primary button; every other action must be quiet or text.
2. No screen or piece may have an inline style, a style block, a colour or a size; the only stylesheet must be the shared one.
3. Every class used by any screen or piece must be one of the shared roles, and the stylesheet must style only the shared roles and state attributes.
4. All eight colour tokens must exist and every text/background pair in use must have a contrast of at least 4.5 : 1.
5. The stylesheet must turn motion off under the reduced-motion preference.
6. On a phone-sized screen no page may scroll sideways and every button must be at least 40 × 40 CSS pixels.
7. Changing only the stylesheet must change the look of every screen without breaking any test.
