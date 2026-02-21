# Zest Content Creator

This is a Zest content creation workspace. Use Claude Code to build interactive content for Canvas LMS.

## Quick Start

- Use `/zest-new` to create new interactive content
- Use `/zest-build` to validate and package content as a `.zest` file for upload

## Conventions

- Content goes in the `content/` directory (one subdirectory per activity)
- Each activity needs at minimum an `index.html` file
- Include a `zest.json` spec manifest with metadata, grading mode, and configuration
- Graded content also needs a `review.html` for SpeedGrader
- All CSS and JS must be inline in the HTML file (self-contained)
- The only external script is the Zest bridge: `https://lti.testyturtle.dev/public/zest-bridge.js`

## Important Canvas Constraints

- **NEVER use `confirm()`, `alert()`, or `prompt()`** — they are silently blocked in cross-origin iframes
- Use the two-click confirmation pattern instead (see content-patterns skill)
- Canvas CSS prevents iframe resize for assignment tools — content scrolls within the iframe

## Grading Modes

- **Auto-graded**: Use `Zest.submitScore(score, options)` — score appears in gradebook immediately
- **Teacher-graded**: Use `Zest.submitWork(options)` — teacher reviews and grades in SpeedGrader
- **No grading**: No submission calls needed

## State Persistence

For activities that take more than a few minutes, implement state persistence:
- `Zest.saveState(data)` — saves student progress
- `Zest.loadState()` — restores previous progress
- `Zest.clearState()` — resets (for "Start Over" button)

## New Features

### Spec File (`zest.json`)
Optional manifest that auto-configures content on upload. Declares grading mode, parameters, answer keys, and sandbox permissions.

### Answer Keys
Store answer keys in a separate JSON file, declared in `zest.json`. Extracted to `.secure/` on upload — never accessible to students. Teachers see them in SpeedGrader via `Zest.getAnswerKey()`.

### Per-Placement Parameters
Define configurable settings in `zest.json` (difficulty, time limit, etc.). Teachers set values per-assignment. Content reads them via `Zest.getParameter(key, default)`.

### Configurable Sandbox
Declare iframe sandbox permissions in `zest.json`. Admins can relax or tighten per-content.

## Workflow

1. Describe what you want to build to Claude
2. Claude creates `index.html`, `review.html` (if graded), `zest.json`, and optionally `answer-key.json`
3. Run `/zest-build` to validate and package as a `.zest` file
4. Upload the `.zest` file to Zest in Canvas
5. Test in Canvas
