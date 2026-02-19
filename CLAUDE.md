# ZipEmbed Content Creator

This is a ZipEmbed content creation workspace. Use Claude Code to build interactive content for Canvas LMS.

## Quick Start

- Use `/zipembed-new` to create new interactive content
- Use `/zipembed-build` to validate and package content as a zip for upload

## Conventions

- Content goes in the `content/` directory (one subdirectory per activity)
- Each activity needs at minimum an `index.html` file
- Graded content also needs a `review.html` for SpeedGrader
- All CSS and JS must be inline in the HTML file (self-contained)
- The only external script is the ZipEmbed bridge: `https://lti.testyturtle.dev/public/zipembed-bridge.js`

## Important Canvas Constraints

- **NEVER use `confirm()`, `alert()`, or `prompt()`** — they are silently blocked in cross-origin iframes
- Use the two-click confirmation pattern instead (see content-patterns skill)
- Canvas CSS prevents iframe resize for assignment tools — content scrolls within the iframe

## Grading Modes

- **Auto-graded**: Use `ZipEmbed.submitScore(score, options)` — score appears in gradebook immediately
- **Teacher-graded**: Use `ZipEmbed.submitWork(options)` — teacher reviews and grades in SpeedGrader
- **No grading**: No submission calls needed

## State Persistence

For activities that take more than a few minutes, implement state persistence:
- `ZipEmbed.saveState(data)` — saves student progress
- `ZipEmbed.loadState()` — restores previous progress
- `ZipEmbed.clearState()` — resets (for "Start Over" button)

## Workflow

1. Describe what you want to build to Claude
2. Claude creates `index.html` (and `review.html` if graded) in `content/`
3. Run `/zipembed-build` to validate and package as a zip
4. Upload the zip to ZipEmbed in Canvas
5. Test in Canvas
