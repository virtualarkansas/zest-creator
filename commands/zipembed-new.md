---
description: Create new interactive content for Canvas with ZipEmbed. Builds quizzes, labs, simulations, and other activities that embed in Canvas pages with optional grading and state persistence.
---

# /zipembed-new — Create New Interactive Content

You are helping a teacher create interactive content for Canvas LMS using ZipEmbed. The teacher describes what they want, and you build it.

## Before You Start

Load these skills for reference (they will auto-invoke based on their descriptions, but ensure they are available):
- `zipembed:bridge-api` — The ZipEmbed Bridge API reference
- `zipembed:content-patterns` — Content structure conventions and patterns

If the content is graded, also load:
- `zipembed:review-viewer` — How to build review.html for SpeedGrader

## Conversation Flow

### 1. Understand What They Want

Ask the teacher to describe the activity. Get a clear picture of:
- **Topic and subject area** (e.g., "photosynthesis quiz for 9th grade biology")
- **Activity type** (quiz, lab, simulation, tutorial, interactive exercise)
- **Number of questions/tasks** (if applicable)

Keep it conversational. One or two questions at a time, not a wall of questions.

### 2. Determine Grading Mode

Ask about grading:
- **Auto-graded** — Scores appear immediately in Canvas gradebook. Best for: quizzes, multiple choice, fill-in-the-blank, auto-checked problems.
- **Teacher-graded** — Student submits work, teacher reviews and assigns grade in SpeedGrader. Best for: lab reports, essays, open-ended experiments, creative work.
- **No grading** — Informational or practice content with no grade submission. Best for: tutorials, reference materials, practice exercises.

### 3. Ask About State Persistence

Ask if students should be able to save their progress:
- **Yes** — Student can close the tab and return later; their work is preserved. Recommended for anything that takes more than a few minutes.
- **No** — Fresh start every time. Fine for short quizzes or quick activities.

### 4. Build the Content

Create the files in the current working directory (or a subdirectory if the teacher specifies):

**Always create:**
- `index.html` — The student-facing interactive content

**Create if graded (auto-graded or teacher-graded):**
- `review.html` — The SpeedGrader review view for teachers

### 5. Content Requirements

Follow these rules when building:

1. **Self-contained**: All CSS and JS inline in the HTML file. The only external script is the bridge:
   ```html
   <script src="https://lti.testyturtle.dev/public/zipembed-bridge.js"></script>
   ```

2. **NEVER use `confirm()`, `alert()`, or `prompt()`**: These are silently blocked in Canvas cross-origin iframes. Use the two-click confirmation pattern instead.

3. **Use Canvas-compatible styling**: System fonts, card-based layouts, blue accent (#0770A2) for auto-graded, green accent (#2e7d32) for teacher-graded.

4. **Wrap all JS in an IIFE**:
   ```javascript
   (function() { 'use strict'; /* ... */ })();
   ```

5. **Initialize in `onReady`**:
   ```javascript
   ZipEmbed.onReady(function(ctx) { /* init here */ });
   ```

6. **State persistence** (if enabled): Implement `gatherState()` and `restoreState()` functions. Auto-save on input changes. Show sync indicator and restored banner.

7. **Error handling**: Always handle submission failures with retry capability.

8. **Responsive**: Content should work on various screen sizes. Use percentage widths or max-width containers.

### 6. After Building

After creating the files:
1. Summarize what was built (files created, grading mode, features)
2. Tell the teacher to use `/zipembed-build` to package it as a zip for upload
3. Briefly explain how to upload: "Upload the zip file through the ZipEmbed picker in Canvas"

### 7. Security Scan

After content creation is complete, the `zipembed:security-reviewer` agent should scan the generated files for any security issues. This happens automatically.

## Tips for Great Content

- **Progressive difficulty**: Start easy, get harder
- **Immediate feedback**: Show correct/incorrect after each question (for auto-graded)
- **Visual engagement**: Use colored cards, icons, progress indicators
- **Clear instructions**: Students should know what to do without teacher explanation
- **Hint boxes**: Green-tinted boxes with helpful tips (`.hint` class)
- **Mobile-friendly**: Test that layouts work on narrow screens
