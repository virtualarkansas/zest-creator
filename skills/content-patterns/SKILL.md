---
name: content-patterns
description: Zest content structure patterns and conventions. Use when creating HTML/JS/CSS interactive content for Canvas, building quizzes, labs, simulations, or any embeddable activity. Covers file structure, state persistence patterns, grading integration, and Canvas iframe constraints.
user-invocable: false
---

# Zest Content Patterns

## File Structure

Every Zest interactive has this structure:

```
my-content/
  index.html         # Student view (REQUIRED)
  review.html        # Teacher SpeedGrader view (required if graded)
  zest.json          # Spec manifest (optional, auto-configures on upload)
  answer-key.json    # Answer key (optional, declared in zest.json, stored securely)
```

Content is self-contained — all CSS and JS are inline in the HTML file. The only external dependency is the bridge script.

## Spec File (`zest.json`)

Optional manifest at the zip root that auto-configures content on upload. If present, its values are used as defaults (the picker UI can still override).

```json
{
  "name": "Photosynthesis Quiz",
  "version": "1.0.0",
  "description": "Auto-graded quiz on photosynthesis",
  "author": "Jane Teacher",
  "grading": "auto",
  "mainFile": "index.html",
  "reviewFile": "review.html",
  "answerKey": "answer-key.json",
  "parameters": {
    "difficulty": {
      "type": "select",
      "label": "Difficulty Level",
      "options": ["easy", "medium", "hard"],
      "default": "medium"
    },
    "timeLimit": {
      "type": "number",
      "label": "Time Limit (minutes)",
      "min": 1,
      "max": 120,
      "default": 30
    }
  },
  "sandbox": {
    "allowScripts": true,
    "allowSameOrigin": true,
    "allowPopups": true,
    "allowForms": true,
    "allowModals": false,
    "allowTopNavigation": false
  }
}
```

**Key fields:**
- `grading`: `"auto"` | `"teacher"` | `"none"` — sets the default grading mode
- `mainFile`: entry point HTML file (default: `index.html`)
- `reviewFile`: SpeedGrader view (default: auto-detected `review.html`)
- `answerKey`: path to answer key file — extracted to `.secure/` directory (never served to students)
- `parameters`: configurable per-placement values — teachers set these when embedding
- `sandbox`: iframe sandbox permissions — controls what the content can do

## HTML Template

Every index.html follows this skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Activity Title</title>
  <script src="https://lti.testyturtle.dev/public/zest-bridge.js"></script>
  <style>
    /* All CSS inline */
  </style>
</head>
<body>
  <!-- Content here -->
  <script>
    (function() {
      'use strict';
      // All JS in an IIFE
      Zest.onReady(function(ctx) {
        // Initialize content after LTI context received
      });
    })();
  </script>
</body>
</html>
```

## Canvas-Compatible Styling

Use system fonts and clean card-based layouts:

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  padding: 24px;
  background: #f8f9fa;
  color: #2d3b45;
}
.card {
  background: white;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 16px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.08);
}
```

**Color conventions**:
- Auto-graded content: Blue accent `#0770A2` (Canvas blue)
- Teacher-graded content: Green accent `#2e7d32`
- Error states: Red `#c0392b`
- Success states: Green `#27ae60`

## State Persistence Pattern

For any content where students should be able to save progress:

### 1. Gather/Restore Functions

```javascript
function gatherState() {
  return {
    answers: { q1: getSelectedValue('q1'), q2: getSelectedValue('q2') },
    currentPhase: currentPhase,
    submitted: isSubmitted
  };
}

function restoreState(state) {
  if (!state) return false;
  if (state.answers) {
    setSelectedValue('q1', state.answers.q1);
    setSelectedValue('q2', state.answers.q2);
  }
  if (state.currentPhase) goToPhase(state.currentPhase);
  if (state.submitted) disableSubmitButton();
  return true;
}
```

### 2. Auto-Save on Input Changes

```javascript
// Immediate save for discrete inputs (radio, select, checkbox)
document.querySelectorAll('input[type="radio"]').forEach(function(r) {
  r.addEventListener('change', function() {
    Zest.saveState(gatherState());
  });
});

// Debounced save for text inputs (1 second delay)
var saveTimeout = null;
document.querySelectorAll('textarea, input[type="text"]').forEach(function(el) {
  el.addEventListener('input', function() {
    if (saveTimeout) clearTimeout(saveTimeout);
    saveTimeout = setTimeout(function() {
      Zest.saveState(gatherState());
    }, 1000);
  });
});
```

### 3. Load and Restore in onReady

```javascript
Zest.onReady(function(ctx) {
  Zest.loadState().then(function(state) {
    if (state) {
      var restored = restoreState(state);
      if (restored) {
        document.getElementById('restored-banner').classList.add('visible');
      }
    }
  });
});
```

### 4. Sync Status Indicator

```javascript
Zest.onSyncStatus(function(status) {
  document.getElementById('sync-dot').className = 'sync-dot ' + status;
  var labels = { synced: 'Saved', dirty: 'Unsaved changes', syncing: 'Saving...', error: 'Save failed' };
  document.getElementById('sync-text').textContent = labels[status] || status;
});
```

## Two-Click Reset Pattern

**NEVER use `confirm()`, `alert()`, or `prompt()`** — they are silently blocked in cross-origin iframes (Canvas). Use this two-click pattern instead:

```javascript
var resetPending = false;

function doReset() {
  if (!resetPending) {
    resetPending = true;
    resetBtn.textContent = 'Click again to confirm reset';
    resetBtn.style.background = '#fdf0ef';
    resetBtn.style.borderColor = '#c0392b';
    resetBtn.style.color = '#c0392b';
    setTimeout(function() {
      resetPending = false;
      resetBtn.textContent = 'Start Over';
      resetBtn.style.background = '';
      resetBtn.style.borderColor = '';
      resetBtn.style.color = '';
    }, 4000);
    return;
  }
  resetPending = false;
  Zest.clearState();
}
```

## Parameter-Driven Content

When `zest.json` declares parameters, content can adapt per-placement:

```javascript
Zest.onReady(function(ctx) {
  var difficulty = Zest.getParameter('difficulty', 'medium');
  var timeLimit = Zest.getParameter('timeLimit', 30);

  if (difficulty === 'hard') {
    enableBonusQuestions();
    shuffleOptions();
  }

  if (timeLimit > 0) {
    startTimer(timeLimit * 60);
  }
});
```

## Answer Key Pattern

For auto-graded content, declare the answer key in `zest.json` and store it in a separate JSON file:

**zest.json:**
```json
{ "answerKey": "answer-key.json", "grading": "auto" }
```

**answer-key.json:**
```json
{ "q1": "b", "q2": "c", "q3": "a" }
```

In `review.html`, teachers can see the answer key:

```javascript
if (Zest.isReviewMode()) {
  var key = Zest.getAnswerKey();
  if (key) {
    // Highlight correct answers in the review view
  }
}
```

The answer key file is extracted to `.secure/` during upload and is **never served to students**.

## Example Content

For detailed working examples of these patterns:
- [Auto-graded quiz pattern](auto-graded-example.md) — radio buttons, auto-scoring, submitScore
- [Teacher-graded lab pattern](teacher-graded-example.md) — multi-phase, textarea inputs, submitWork, simulation
