# 13. Figures

## 13.1 Purpose and principle

Many geometry, coordinate and measurement problems cannot be stated well without a picture: a right triangle with two sides given, a tangent to a circle at a point, a parabola on axes. A wrong picture is worse than none: a student who trusts a mislabelled side learns the wrong thing, and the diagnosis records a gap that is not there.

So in ThyGap:

- **A figure is data, never a model image.** A figure is a **FigureSpec**: a short description of points, segments and shapes with their labels. No model ever draws pixels for a student.
- **The app draws every figure itself**, with its own renderer, the same way every time.
- **Every model-written figure passes the figure gate** before a student sees it: it must be well formed, its numbers must match what it draws, and it must name the same points as the problem text.

Which concepts get figure problems is decided by the concept map: only concepts where a figure is needed or helpful (chapter 6). A composed problem asks for a figure when its main concept or a support concept is marked "figure needed" or "figure helpful" (12.4).

## 13.2 What a FigureSpec holds

| part | meaning |
|---|---|
| kind | a plain figure (no axes) or a coordinate figure (drawn on coordinate axes) |
| axes | for a coordinate figure, the x range and the y range shown |
| points | each with a one-capital-letter id, its position, and whether it is shown (a dot and its name) or only a construction point |
| segments | between two points, solid or dashed |
| polygons | a list of point ids drawn as a closed shape |
| circles | a centre point and either a radius or a point the circle passes through |
| right-angle marks | the small square at a vertex between two rays |
| angle marks | an arc at a vertex between two rays |
| equal-length ticks | the same number of ticks on segments of equal length |
| labels | a text attached to a point, a segment, an angle or a circle. The text is a number, a coordinate pair (points only) or one lower-case variable |
| lines | infinite lines through two points, or given by slope and intercept |
| graphs | curves y = expression in x (linear or quadratic) |
| relations | facts the figure claims: a line tangent to a circle, two circles tangent, two lines parallel or perpendicular |
| asked label | which label the question asks for (the unknown `x` on a side) |

## 13.3 Drawing rules

**Fitting into the box**

- Every figure is drawn in a box of the same width, so every figure in a quiz looks the same size. The height follows the figure and is never more than the width.
- One scale for both directions (a circle stays round, a right angle stays right), as large as fits. Nothing is cut off.

**What is drawn**

- A coordinate figure gets both axes, the axis names and the origin. **The figure never shows a number the problem does not give**, on the axes or anywhere else.
- Shapes, marks and labels are drawn plainly in one ink colour on a white background; a shown point gets a dot and its name.

**Label placement**

Labels sit next to what they name and never overlap lines, dots or each other, or a student misreads the figure.

**Safe and self-contained**

- The renderer produces one self-contained SVG with an accessible name. Every text in it is escaped; it contains no script, no link and nothing that loads from outside.
- **The same FigureSpec always gives exactly the same SVG.**
- The renderer can report where it placed things, so tests will be able to prove that nothing overlaps.
- Each figure has a one-line description in the student's language listing its parts and labels. It is the figure's accessible name.

## 13.4 The figure gate

A model writes the FigureSpec of a composed problem together with the problem text. The figure gate is part of the composed-problem gate and checks the figure before the problem can be read back, stored or served. Its errors go back to the model as fix requests within the composer's one fix try. A problem whose figure still fails is dropped and never reaches a student. A problem whose plan needs no figure must carry none.

It rejects:

1. **A missing figure** when the plan needs one.
2. **A malformed figure**, such as a reference to a point that does not exist.
3. **A wrong measure**: a number on a side or angle that differs from the drawn length or angle; a number label that is not one of the problem's given values; an asked label that does not exist; an asked label whose drawn size is not the answer.
4. **Names that disagree**: a point named in the problem text that is not in the figure; a shown point the problem text does not name.

## 13.5 Module boundaries

| module | responsibility | depends on |
|---|---|---|
| figure frame | Holds the shared geometry rules: the box, the scale, where things land, and which numbers a figure may show. | the shared figure types |
| label placer | Places every label next to what it names, clear of lines, dots and other labels. | figure frame |
| figure renderer | Draws a FigureSpec as safe SVG with its one-line description. | figure frame, label placer, calculator (graph values) |
| figure gate | Checks a model-written figure against its problem. | figure frame |
| figure view | Shows a figure on screen in the student's language; it is the only place that inserts figure markup, and that markup only comes from the figure renderer. | figure renderer, the language setting |

## 13.6 Figure test coverage

The tests must cover triangle, circle-tangent and graph cases, each checked by computation, plus broken figures the gate must refuse.

## 13.7 Acceptance criteria

1. Every figure must be drawn at the same width, never taller than wide, with one scale in both directions and every point and label inside the box.
2. In the drawn layout no two labels may overlap, no label may cross a line it does not name, and no point name may cover another dot.
3. A coordinate figure must show a tick number only when that number appears in the problem text; otherwise its ticks must have no numbers.
4. The same FigureSpec must always give the identical SVG; a label containing markup must appear only escaped, and the SVG must contain no script, link or external reference.
5. A model-written figure whose labelled side length differs from its drawn length, whose asked label's size is not the answer, or whose points differ from those named in the text must be rejected and never shown to a student.
6. The figure's accessible name must list its points, sides, right angles and labels, and change language with the student's language setting.
