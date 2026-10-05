# Repository standard and development pipeline

This workflow covers the whole project: define it, establish its repository, build a running application, then repeatedly implement and test features. **README.md holds the shared plan and instructions for you and AI.**

## Pipeline

**Initial setup: 1 → 2 → 3**  
**Feature loop: 4 → 5 → 6 → 4**

### 1 Define the project
Write the product’s purpose, users, first usable version, later priorities, and constraints in README.md.

### 2 Establish the repository
Choose the toolchain, apply the layout below, and create the required project files.

#### Repository layout

```text
project/
├── README.md
├── AGENTS.md
├── STRUCTURE.md
├── TODO.md
├── .gitignore
├── <dependency manifest>
├── <dependency lockfile>
├── src/
│   ├── app/               entry points and wiring
│   ├── features/          code grouped by feature
│   ├── shared/            genuinely reused code
│   └── infrastructure/    storage, network, external integrations
├── tests/                 checks and test data
├── data/                  datasets and generated data, when needed
├── docs/
│   └── PROJECT_DOC.md     whole-project technical reference
├── assets/                static files, when needed
├── config/                configuration, when needed
└── .env.example           settings, when needed; no secrets
```

Application and utility code belongs in src; tests stay in root tests/. Require the five Markdown files, `.gitignore`, and the chosen toolchain’s dependency manifest and lockfile. Create directories when used. Keep large or sensitive data out of Git.

#### File responsibilities

1. **README.md:** project purpose/behavior, full pipeline, standards, run/test commands, and implemented-feature index and planned-feature index. Run commands at the very top just below the overview description.
2. **AGENTS.md:** directs AI to README.md, STRUCTURE.md, and its TODO item; tool-specific instructions only
3. **STRUCTURE.md:** current repository map and responsibilities
4. **TODO.md:** feature specifications, completion status, and bugs found
5. **docs/PROJECT_DOC.md:** whole-project technical reference, linked from README.md

Link instructions rather than duplicating them.

#### Coding and handoff standards

- Clear names, focused functions, explicit errors, clear responsibilities; document meaningful types
- Every file starts with a header containing Purpose: (1–3 sentences) and File Dependencies: (referenced project files, or None)
- Leave exactly two blank lines after the header before the code
- Put one complete, grammatical sentence immediately above every function, constructor, helper, and event handler, explaining what it does
- Inside functions, use a few short comments describing operations, such as rejecting NaN or initializing variables
- Match checks to risk; an AI’s assertion is not verification
- If interrupted, leave status/next steps; keep unfinished items open
- Summarize results, checks, uncertainty, and decisions; expand when useful

##### Code Standard Examples:

##### Java

File: src/features/range/Range.java

```java
/*
Purpose: This file clamps finite numbers to inclusive bounds without I/O or shared state.
File Dependencies: None.
*/


/** This class provides validated numeric range operations. */
public final class Range {
    // This constructor prevents instances because the class provides only a static operation.
    private Range() {}

    /** This method clamps the value to inclusive bounds or throws IllegalArgumentException for invalid inputs. */
    public static double clamp(double value, double minimum, double maximum) {
        // Reject non-finite inputs and reversed bounds.
        if (!Double.isFinite(value) || !Double.isFinite(minimum)
                || !Double.isFinite(maximum) || minimum > maximum) {
            throw new IllegalArgumentException("Require finite inputs and minimum <= maximum.");
        }
        // Clamp values outside the range.
        if (value < minimum) return minimum;
        if (value > maximum) return maximum;
        return value;
    }
}
```
##### C

File: src/features/range/range.c

```c
/*
Purpose: This file clamps finite numbers to inclusive bounds using a caller-provided output.
File Dependencies: None (project files); standard headers: stdbool.h, stddef.h, math.h.
*/


#include <stdbool.h>
#include <stddef.h>
#include <math.h>

/** This function writes the clamped value to a writable result pointer and returns true, or leaves the output unchanged and returns false for invalid input. */
bool clamp(double value, double minimum, double maximum, double *result) {
    // Reject missing output, non-finite inputs, and reversed bounds.
    if (result == NULL || !isfinite(value) || !isfinite(minimum)
            || !isfinite(maximum) || minimum > maximum) {
        return false;
    }
    // Clamp values outside the range.
    if (value < minimum) value = minimum;
    else if (value > maximum) value = maximum;
    *result = value;
    return true;
}
```
##### JavaScript

File: src/features/range/range.mjs

```javascript
/*
Purpose: This file clamps finite numbers to inclusive bounds without DOM access, I/O, or shared state.
File Dependencies: None.
*/


/** This function clamps the value to inclusive bounds or throws RangeError for non-finite inputs, non-numbers, or reversed bounds. */
export function clamp(value, minimum, maximum) {
    // Reject non-finite inputs and reversed bounds.
    if (!Number.isFinite(value) || !Number.isFinite(minimum)
            || !Number.isFinite(maximum) || minimum > maximum) {
        throw new RangeError("Require finite inputs and minimum <= maximum.");
    }
    // Clamp values outside the range.
    if (value < minimum) return minimum;
    if (value > maximum) return maximum;
    return value;
}
```
##### HTML interface

File: src/app/index.html

HTML provides the form; its module calls the same JavaScript function above. Empty numeric fields become NaN and receive the same validation error.

```html
<!--
Purpose: This file provides range inputs and displays the clamped value or validation error.
File Dependencies: ../features/range/range.mjs.
-->


<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Clamp a number</title>
</head>
<body>
  <main>
    <h1>Clamp a number</h1>
    <form id="range-form" novalidate>
      <label for="value">Value</label>
      <input id="value" name="value" type="number" step="any" value="125" required>
      <label for="minimum">Minimum</label>
      <input id="minimum" name="minimum" type="number" step="any" value="0" required>
      <label for="maximum">Maximum</label>
      <input id="maximum" name="maximum" type="number" step="any" value="100" required>
      <button type="submit">Clamp</button>
    </form>
    <output id="result" for="value minimum maximum" aria-live="polite">Ready</output>
  </main>
  <script type="module">
    import { clamp } from "../features/range/range.mjs";

    // Initialize the form and result elements.
    const form = document.querySelector("#range-form");
    const result = document.querySelector("#result");
    // This handler displays the clamped value or validation error when the form is submitted.
    form.addEventListener("submit", (event) => {
      // Keep the result on this page.
      event.preventDefault();
      // This helper reads a named form field as a number.
      const read = (name) => form.elements.namedItem(name).valueAsNumber;
      try {
        // Calculate and display the result.
        result.textContent = String(clamp(read("value"), read("minimum"), read("maximum")));
      } catch (error) {
        // Display the validation error.
        result.textContent = error.message;
      }
    });
  </script>
</body>
</html>
```
##### Handoff example

Illustrative handoff for this sample, not a report about your project:

- Changed: added range validation and a form that displays the result or error
- Checked: normal values, boundaries, equal/reversed bounds, and non-finite inputs
- Remaining: real-browser interaction and accessibility testing
- Documentation: record the input contract, error behavior, and owning files in PROJECT_DOC.md

### 3 Initialize the application
Build a minimal running interface with one or a few pages; establish and verify setup, formatting, build, and test commands.

### 4 Specify the next feature
Review TODO.md and reported bugs; choose the next item. Specify its behavior, edge cases, dependencies, rough code approach, and completion checks.

#### TODO.md format

Keep this labeled example first, then two blank lines before the checklist. Bugs reported by you or AI need only descriptions.

Mark any field or detail **“Up to AI”** to delegate that choice within project constraints. AI determines routine delegated choices without asking again and records the outcome in the implementation or documentation as relevant. Blank fields are not delegation.

```markdown
## Format example (not an active task)

- [ ] Search note titles
  - Behavior: Filter titles as typed, ignoring case. Include archived
    notes. Empty input shows all notes; no matches shows a message.
    Never change saved notes.
  - Dependencies: Existing notes list and note store
  - Approach: Add input to the notes page; keep matching with the
    feature and reuse the existing note store.
  - Checks: Known titles, letter case, archived notes, empty input,
    no matches, and the complete browser interaction.


## Feature checklist

- [ ] Feature title
  - Behavior:
  - Dependencies:
  - Approach: Up to AI
  - Checks:

## Bugs found

- Bug description

## Up to AI

The user can mark any field or detail “Up to AI” to delegate that choice
within project constraints. AI determines routine delegated choices
without asking again and records the outcome in the implementation or
documentation as relevant. Blank fields are not delegation.
```

### 5 Initialize AI and implement
AI reads the project instructions and selected item, inspects relevant code, tests, and unfinished changes, then states its plan and implements within scope.

### 6 Test and check off
Test the complete feature against independently chosen expected results. Once tested to work, update affected documentation, check off the item, and return to step 4; failed or missing required checks keep it open.

#### Documentation contract

Use this outline for docs/PROJECT_DOC.md. Replace the placeholders and illustrative feature with the actual project. Update affected entries at feature completion.

```markdown
# [Project name] Technical Documentation

## Project overview
- What it is: [Purpose and intended users]
- What it does: [Implemented capabilities and scope]

## Architecture and data flow
- Components: [Each component's responsibility and source location]
- Data flow: [How inputs are processed, stored, and turned into outputs]
- Dependencies: [External systems, libraries, and required formats]
- Constraints: [Important assumptions and design decisions]

## Implemented features

### Clamp a number (format example only)
- What it does: Limits a number to an inclusive range.
- Inputs: x is the value; lower and upper are the bounds.
- Output: y is the clamped value; all four use the same units.
- Equation: y = min(max(x, lower), upper).
- How it works: Reject invalid inputs. Return lower if x < lower,
  upper if x > upper, otherwise x.
- Code: src/features/range/range.mjs, clamp(value, minimum, maximum).
- Interface: src/app/index.html reads the inputs, calls clamp,
  and displays the result or error.
- Assumptions: Inputs are finite numbers and lower <= upper.
- Errors: Non-numbers, non-finite inputs, or reversed bounds throw RangeError.
- Limits: Clamps one number; performs no unit conversion or storage.
```
