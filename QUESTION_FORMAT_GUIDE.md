# MonoFlash Question Authoring Guide

This guide explains how to create question data for MonoFlash. It is written for humans and for other AIs that generate importable study content.

The safest default is the plain-text block format pasted into the **Data** tab. MonoFlash also accepts JSON imports, but plain text is easier to write and review.

## 1. Choose the lesson type first

Every lesson has a study type:

| Lesson type | Required line format | Study behavior |
|---|---|---|
| Flashcards | `Question -!- Answer` | The question appears first; the answer appears after flipping. |
| SAQ | `Question -!- Answer` | The question appears first; the answer stays hidden until the learner taps to reveal it. |
| MCQ | `Question -!- Option 1 -!- Option 2 -!- Option 3 -!- Option 4 -!- Correct Answer` | Four options are shown and the learner selects one. The correct option is graded automatically. |

The delimiter is exactly:

```text
-!-
```

It is three characters: a hyphen, an exclamation mark, and a hyphen. Keep it exactly as written.

A two-part line is interpreted according to the destination lesson's type. The same `Question -!- Answer` syntax therefore works for both regular flashcards and SAQs.

## 2. Basic block format

Use a `##` header to select a lesson. Put one card on each following line.

```text
## Lesson Name
Question 1 -!- Answer 1
Question 2 -!- Answer 2
Question 3 -!- Answer 3
```

Example:

```text
## Biology — Cell Structure
What is the powerhouse of the cell? -!- The mitochondrion
What controls what enters and leaves the cell? -!- The cell membrane
What contains the cell's genetic material? -!- The nucleus
```

### Lesson headers

A lesson header is any trimmed line that starts with `##`:

```text
## Biology — Cell Structure
```

The text after the two hashes becomes the lesson name. Leading and trailing spaces are removed, so these are equivalent:

```text
## Biology — Cell Structure
##Biology — Cell Structure
```

Use a different header whenever cards should go into a different lesson:

```text
## Kinematics
What is velocity? -!- Displacement per unit time

## Dynamics
What is Newton's second law? -!- F = ma
```

A blank line is optional and ignored. It is recommended because it makes large imports easier to read.

If no `##` header is supplied, MonoFlash places cards into the currently selected lesson. If no lesson is selected, the parser uses `Lesson 1`.

## 3. Flashcard format

A regular flashcard has exactly one question and one answer conceptually:

```text
Question -!- Answer
```

Example:

```text
## Physics — Motion
What is the formula for average speed? -!- $\text{speed} = \dfrac{\text{distance}}{\text{time}}$
What is acceleration? -!- The rate of change of velocity with respect to time.
```

The parser trims whitespace around each part. These forms are equivalent:

```text
What is acceleration? -!- The rate of change of velocity.
What is acceleration?    -!-    The rate of change of velocity.
```

For reliable generation, use one complete card per physical line. Do not wrap one card across multiple lines.

## 4. SAQ format

SAQs use the same two-part syntax as flashcards:

```text
Question -!- Answer
```

The destination lesson must be created as an **SAQ** lesson. The parser then stores the card with `type: "saq"`, and the app hides the answer until the learner taps **Tap to reveal answer**.

Example:

```text
## Organic Chemistry — Short Answer
What is an alkane? -!- A saturated hydrocarbon containing only carbon-carbon single bonds.
Why are alkenes more reactive than alkanes? -!- Their carbon-carbon double bond contains a reactive pi bond.
```

Do not add `SAQ`, `Answer:`, or another label inside the syntax unless you want that label to appear as part of the answer:

```text
What is an alkane? -!- A saturated hydrocarbon containing only carbon-carbon single bonds.
```

## 5. MCQ format

An MCQ line must contain exactly six segments separated by `-!-`:

```text
Question -!- Option 1 -!- Option 2 -!- Option 3 -!- Option 4 -!- Correct Answer
```

The six segments are:

1. The question.
2. Option 1.
3. Option 2.
4. Option 3.
5. Option 4.
6. The correct answer.

Example:

```text
## Physics — Units
Which quantity is measured in newtons? -!- Energy -!- Force -!- Power -!- Pressure -!- Force
```

The final `Correct Answer` must be textually identical to one of the four options after trimming surrounding whitespace. Use the exact same spelling, capitalization, symbols, and punctuation to avoid ambiguity.

Recommended:

```text
Which planet is closest to the Sun? -!- Venus -!- Earth -!- Mercury -!- Mars -!- Mercury
```

Risky because the answer does not exactly match the option's capitalization:

```text
Which planet is closest to the Sun? -!- Venus -!- Earth -!- Mercury -!- Mars -!- mercury
```

The current parser accepts this row as an MCQ because it checks that all six fields are non-empty, but the study UI compares the selected option with the stored correct answer. Therefore, always copy the correct option exactly into the final field.

Each MCQ should have four non-empty options. Avoid duplicate options because they make grading ambiguous even if the parser accepts them.

## 6. Complete examples

### Flashcard lesson

```text
## Mathematics — Quadratics
What is the standard form of a quadratic equation? -!- $ax^2 + bx + c = 0$
What is the discriminant of $ax^2+bx+c=0$? -!- $\Delta = b^2 - 4ac$
What does a negative discriminant indicate over the real numbers? -!- The equation has no real roots.
```

### SAQ lesson

Create the lesson as **SAQ**, then import:

```text
## Mathematics — Explain the Concept
Why does a quadratic with a positive discriminant have two distinct real roots? -!- Its graph intersects the x-axis at two different points.
What does the vertex represent on a parabola? -!- The maximum or minimum point of the quadratic function.
```

### MCQ lesson

Create the lesson as **MCQ**, then import:

```text
## Mathematics — Quick Check
What is the derivative of $x^2$? -!- $x$ -!- $2x$ -!- $x^3$ -!- $2$ -!- $2x$
If $x=3$, what is $x^2+1$? -!- 7 -!- 9 -!- 10 -!- 12 -!- 10
```

### Multiple lessons in one import

```text
## Biology — Cells
What is the basic unit of life? -!- The cell
What organelle produces most cellular ATP? -!- The mitochondrion

## Biology — Genetics
What molecule carries genetic information? -!- DNA
What is an allele? -!- An alternative form of a gene.
```

For MCQ lessons, every card line in that lesson should use the six-part MCQ format. For SAQ lessons, use the two-part format.

## 7. LaTeX and rich content

Question and answer text can include LaTeX. MonoFlash renders these delimiters with KaTeX:

```text
Inline math: $x^2 + y^2 = z^2$
```

```text
Display math: $$\dfrac{-b \pm \sqrt{b^2-4ac}}{2a}$$
```

Also supported:

```text
Inline: \(F = ma\)
Display: \[E = mc^2\]
```

Example card:

```text
## Physics — Equations
What is Newton's second law? -!- $F = ma$
What is the quadratic formula? -!- $$x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$
```

When generating JSON or text programmatically, preserve the backslashes. In a JSON string, backslashes must be escaped:

```json
{
  "q": "What is Newton's second law?",
  "a": "$F = ma$"
}
```

For a literal LaTeX command inside JSON, write for example `"\\dfrac{1}{2}"` in the JSON source.

## 8. Delimiter rules and edge cases

The parser processes each non-empty line independently:

- Lines beginning with `##` change the active lesson.
- Lines without `-!-` are ignored.
- The first segment becomes the question.
- For a six-segment line, MonoFlash attempts MCQ parsing.
- Otherwise, the first segment becomes the question and all remaining segments are joined back together with `-!-` as the answer.
- Empty questions, answers, or MCQ options are skipped.
- Whitespace around segments is removed.
- A blank line is ignored.
- There is no quote or escape syntax for a literal `-!-` inside a question or answer.

Because the parser splits on every `-!-`, do not use the delimiter as ordinary text inside content. For example, this is not safe:

```text
What does the notation -!- mean? -!- It separates fields.
```

If a question or answer needs to explain the delimiter, write it with spaces or code formatting instead:

```text
What symbol separates fields in a MonoFlash import? -!- The three-character sequence hyphen, exclamation mark, hyphen.
```

The parser does preserve extra delimiters in a non-MCQ answer by joining the remaining pieces back together. However, avoiding the delimiter inside content is still recommended because six segments can be interpreted as an MCQ candidate.

## 9. JSON import format

The Data tab accepts JSON as well as block text. Use a JSON array of lesson objects when you need to preserve lesson types explicitly:

```json
[
  {
    "name": "Biology — Cells",
    "type": "flashcard",
    "cards": [
      {
        "q": "What is the basic unit of life?",
        "a": "The cell"
      }
    ]
  },
  {
    "name": "Biology — Short Answer",
    "type": "saq",
    "cards": [
      {
        "q": "Why is the cell membrane selectively permeable?",
        "a": "It allows some substances to pass while restricting others."
      }
    ]
  },
  {
    "name": "Biology — Quick Check",
    "type": "mcq",
    "cards": [
      {
        "q": "Which organelle contains genetic material?",
        "a": "Nucleus",
        "type": "mcq",
        "options": ["Ribosome", "Nucleus", "Cell wall", "Vacuole"],
        "correct": "Nucleus"
      }
    ]
  }
]
```

Supported lesson type values are:

```text
flashcard
saq
mcq
```

For MCQ cards, include all of these fields:

```json
{
  "q": "Question text",
  "a": "Correct option",
  "type": "mcq",
  "options": ["Option 1", "Option 2", "Option 3", "Option 4"],
  "correct": "Correct option"
}
```

The plain-text importer creates the MCQ `a` and `correct` values from the final field automatically.

The app also accepts a legacy flat JSON array such as:

```json
[
  {"q":"What is H2O?","a":"Water"},
  {"q":"What is CO2?","a":"Carbon dioxide"}
]
```

A flat array is added to the currently selected lesson, or to `Lesson 1` when no lesson is selected. Use the lesson-object format for new generated content because it is explicit and less likely to be misrouted.

## 10. How an AI should generate questions

When asked to create MonoFlash content, follow this procedure:

1. Identify the requested lesson type: flashcard, SAQ, or MCQ.
2. Create a clear lesson header using `## Lesson Name`.
3. Put exactly one card on each line.
4. Use `-!-` only as the field separator.
5. For flashcards and SAQs, output exactly one question field and one answer field.
6. For MCQs, output exactly four options and then repeat the correct option verbatim as the sixth field.
7. Keep every field non-empty.
8. Make questions test one idea at a time.
9. Put the answer in the answer field, not in the question.
10. Preserve LaTeX delimiters and backslashes exactly when math is required.
11. Avoid putting `-!-` inside the question or answer.
12. Return only the import block when the user asks for paste-ready content, unless they request explanations too.

### AI output template: flashcards

```text
## [Lesson name]
[Question 1] -!- [Answer 1]
[Question 2] -!- [Answer 2]
```

### AI output template: SAQs

```text
## [SAQ lesson name]
[Short-answer question 1] -!- [Expected answer 1]
[Short-answer question 2] -!- [Expected answer 2]
```

### AI output template: MCQs

```text
## [MCQ lesson name]
[Question 1] -!- [Option 1] -!- [Option 2] -!- [Option 3] -!- [Option 4] -!- [Correct option]
[Question 2] -!- [Option 1] -!- [Option 2] -!- [Option 3] -!- [Option 4] -!- [Correct option]
```

## 11. Quality checklist before importing

Check every generated line:

- Does it contain the exact `-!-` delimiter?
- Does it have a non-empty question?
- Does it have a non-empty answer?
- Is the line on the intended lesson section?
- If it is MCQ, are there exactly four options?
- Is the final MCQ field an exact copy of one option?
- Are the options meaningfully different and not duplicates?
- Is the answer factually correct and specific enough?
- Is the question answerable without guessing the author's intention?
- Are LaTeX delimiters balanced?
- Does the question or answer accidentally contain `-!-` as normal prose?
- Are duplicate questions being avoided? MonoFlash skips a new card when an existing card has the same question, ignoring case and surrounding whitespace.

## 12. Import workflow in the app

1. Create or select the subject and topic.
2. Create a lesson and choose its type: **Flashcards**, **MCQ**, or **SAQ**.
3. Open the **Data** tab.
4. Paste the plain-text block into the import box.
5. Click **Import**.
6. Review the cards in the database list.
7. For MCQs, confirm the correct option is shown as the correct answer.
8. Start the lesson and test a few cards before generating a large batch.

A successful import reports how many cards were added and how many duplicate questions were skipped.
