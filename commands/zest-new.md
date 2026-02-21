---
description: Create new interactive content for Canvas with Zest. Builds quizzes, labs, simulations, and other activities that embed in Canvas pages with optional grading and state persistence.
---

# /zest-new — Create New Interactive Content

You are helping a teacher create interactive content for Canvas LMS using Zest. The teacher describes what they want, and you build it.

## Before You Start

Load these skills for reference (they will auto-invoke based on their descriptions, but ensure they are available):
- `zest:bridge-api` — The Zest Bridge API reference
- `zest:content-patterns` — Content structure conventions and patterns

If the content is graded, also load:
- `zest:review-viewer` — How to build review.html for SpeedGrader

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

### 4. Ask About Parameters (Optional)

If the content could benefit from teacher-configurable settings, offer parameters:
- **Difficulty level** — easy/medium/hard
- **Time limit** — minutes
- **Number of questions** — if the content is a quiz
- **Show hints** — yes/no

Parameters let teachers customize the same content for different assignments without creating multiple versions.

### 5. Build the Content

Create the files in the current working directory (or a subdirectory if the teacher specifies):

**Always create:**
- `index.html` — The student-facing interactive content
- `zest.json` — Spec manifest with metadata, grading mode, and configuration

**Create if graded (auto-graded or teacher-graded):**
- `review.html` — The SpeedGrader review view for teachers

**Create if auto-graded with separate answer key:**
- `answer-key.json` — Answer key file (referenced in `zest.json`, stored securely on upload)

### 6. Content Requirements

Follow these rules when building:

1. **Self-contained**: All CSS and JS inline in the HTML file. The only external script is the bridge:
   ```html
   <script src="https://lti.testyturtle.dev/public/zest-bridge.js"></script>
   ```

2. **NEVER use `confirm()`, `alert()`, or `prompt()`**: These are silently blocked in Canvas cross-origin iframes. Use the two-click confirmation pattern instead.

3. **Use Canvas-compatible styling**: System fonts, card-based layouts, blue accent (#0770A2) for auto-graded, green accent (#2e7d32) for teacher-graded.

4. **Wrap all JS in an IIFE**:
   ```javascript
   (function() { 'use strict'; /* ... */ })();
   ```

5. **Initialize in `onReady`**:
   ```javascript
   Zest.onReady(function(ctx) { /* init here */ });
   ```

6. **State persistence** (if enabled): Implement `gatherState()` and `restoreState()` functions. Auto-save on input changes. Show sync indicator and restored banner.

7. **Error handling**: Always handle submission failures with retry capability.

8. **Responsive**: Content should work on various screen sizes. Use percentage widths or max-width containers.

9. **Always generate `zest.json`**: Include metadata, grading mode, and any parameters:
   ```json
   {
     "name": "Activity Title",
     "version": "1.0.0",
     "grading": "auto",
     "mainFile": "index.html",
     "reviewFile": "review.html"
   }
   ```

10. **Parameters** (if applicable): Use `Zest.getParameter(key, default)` to read teacher-set values. Never hardcode configurable values — make them parameters so teachers can customize per-placement.

11. **Answer keys** (for auto-graded): Store the answer key in a separate `answer-key.json` file and declare it in `zest.json` via `"answerKey": "answer-key.json"`. The server extracts it to `.secure/` — never accessible to students. In `review.html`, use `Zest.getAnswerKey()` to compare against student answers.

### 7. After Building

After creating the files:
1. Summarize what was built (files created, grading mode, features)
2. Tell the teacher to use `/zest-build` to package it as a zip for upload
3. Briefly explain how to upload: "Upload the zip file through the Zest picker in Canvas"

### 8. Security Scan

After content creation is complete, the `zest:security-reviewer` agent should scan the generated files for any security issues. This happens automatically.

## Tips for Great Content

- **Progressive difficulty**: Start easy, get harder
- **Immediate feedback**: Show correct/incorrect after each question (for auto-graded)
- **Visual engagement**: Use colored cards, icons, progress indicators
- **Clear instructions**: Students should know what to do without teacher explanation
- **Hint boxes**: Green-tinted boxes with helpful tips (`.hint` class)
- **Mobile-friendly**: Test that layouts work on narrow screens
