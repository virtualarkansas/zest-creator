---
name: bridge-api
description: ZipEmbed Bridge API reference. Use when writing JavaScript that calls ZipEmbed.submitScore(), ZipEmbed.submitWork(), ZipEmbed.saveState(), ZipEmbed.loadState(), or any other ZipEmbed.* method. Also use when building interactive content for Canvas LTI.
user-invocable: false
---

# ZipEmbed Bridge API

The Bridge API is a JavaScript library that content creators include in their interactive content. It provides a standard interface for communicating with the ZipEmbed LTI wrapper inside Canvas.

## Include

Every content file must include this script tag in `<head>`:

```html
<script src="https://lti.testyturtle.dev/public/zipembed-bridge.js"></script>
```

## Critical Constraints

1. **`confirm()`, `alert()`, `prompt()` are BLOCKED** in cross-origin iframes (Canvas embeds from a different origin). They silently return `false`/`undefined`. Use two-click confirmation patterns instead.
2. **Canvas CSS prevents iframe resize** for assignment external tools. Content scrolls within the iframe. `setHeight()` only works for page embeds.
3. **Content must be self-contained** — all CSS and JS inline or bundled in the zip. The bridge script is the only external dependency.
4. **All methods require context** — most methods only work after `onReady()` fires. Always wrap initialization in `ZipEmbed.onReady()`.

---

## Context Methods

### ZipEmbed.getUser()
Returns the current user's context. Returns `null` if not in an LTI context.
```javascript
const user = ZipEmbed.getUser();
// { id, name, email, roles, courseId, courseTitle, assignmentId }
```

### ZipEmbed.isStudent()
```javascript
if (ZipEmbed.isStudent()) { /* show student view */ }
```

### ZipEmbed.isTeacher()
```javascript
if (ZipEmbed.isTeacher()) { /* show teacher/admin view */ }
```

### ZipEmbed.onReady(callback)
Fires when LTI context is received from the wrapper. If context is already available, fires immediately. **Always use this as your entry point.**
```javascript
ZipEmbed.onReady(function(ctx) {
  console.log('User:', ctx.name);
  console.log('Course:', ctx.courseTitle);
  // Initialize your content here
});
```

### ZipEmbed.isReady()
Returns `true` if context has been received.

---

## Grading Methods

### ZipEmbed.submitScore(score, options) — Auto-Graded

Submits a score that appears immediately in the Canvas gradebook. Use for quizzes, auto-checked exercises, and anything the content can grade itself.

```javascript
const result = await ZipEmbed.submitScore(85, {
  maxScore: 100,           // Optional, default 100
  artifacts: {             // Optional — arbitrary JSON stored for teacher review
    answers: { q1: 'A', q2: 'B', q3: 'C' },
    timing: { duration: 4444 }
  },
  comment: 'Scored 85%'   // Optional
});
// result: { success: true } or { success: false, error: '...' }
```

**Canvas behavior**: Score appears immediately in gradebook as 85/100. Teacher can view artifacts in SpeedGrader via review.html.

### ZipEmbed.submitWork(options) — Teacher-Graded

Submits student work without a score. Gradebook shows "Submitted" and the teacher assigns a grade manually in SpeedGrader.

```javascript
const result = await ZipEmbed.submitWork({
  artifacts: {             // Arbitrary JSON — teacher sees this in review.html
    hypothesis: 'Plants grow faster with more light',
    data: [/* experiment results */],
    analysis: 'The results show...'
  },
  comment: 'Lab completed'  // Optional
});
// result: { success: true } or { success: false, error: '...' }
```

**Canvas behavior**: Gradebook shows "Submitted" with no score. Teacher opens SpeedGrader, sees artifacts in review.html, enters grade manually.

### Grading Mode Comparison

| | submitScore() | submitWork() |
|---|---|---|
| Score | Required (number) | None (null) |
| Gradebook | Score appears immediately | Shows "Submitted" |
| Teacher action | Can review artifacts | Must assign grade |
| Use case | Quiz, auto-check | Lab report, essay, experiment |

---

## Review Mode Methods

### ZipEmbed.isReviewMode()
Returns `true` when the teacher is viewing this content in SpeedGrader.
```javascript
if (ZipEmbed.isReviewMode()) {
  const sub = ZipEmbed.getSubmission();
  // Render student's submitted data for teacher review
}
```

### ZipEmbed.getSubmission()
In review mode, returns the student's submitted data.
```javascript
const sub = ZipEmbed.getSubmission();
// { score, maxScore, artifacts, comment, submittedAt, userId, userName }
// score/maxScore are null for teacher-graded (submitWork) submissions
```

---

## UI Methods

### ZipEmbed.setHeight(pixels)
Reports content height for auto-resize. Sends `lti.frameResize` to Canvas.

**Note**: Only works for **page embeds** (deep link html mode). Has **no effect on assignment external tools** due to Canvas CSS `!important` limitation. For assignments, content scrolls within the iframe.

```javascript
ZipEmbed.setHeight(document.body.scrollHeight + 40);
```

---

## State Persistence Methods

Three-tier storage: Memory (undo/redo) → localStorage (fast/offline) → Server/PostgreSQL (durable/cross-device).

State is keyed by `contentId + assignmentId + userId` — same content in two different assignments = independent saves.

### ZipEmbed.saveState(data)
Save work-in-progress. Writes to localStorage immediately, marks state as dirty for server sync (every 60 seconds). Pushes current state to undo stack and clears redo stack.

```javascript
ZipEmbed.saveState({
  currentPhase: 3,
  answers: { q1: 'a', q2: 'b' },
  hypothesis: 'Plants grow faster with more light'
});
// No return value — fire and forget
```

### ZipEmbed.loadState()
Load saved state. First call fetches from server and compares with localStorage — uses the newer one. Subsequent calls return from localStorage (fast).

```javascript
const state = await ZipEmbed.loadState();
if (state) {
  restoreForm(state);  // Rebuild UI from saved state
}
// Returns: data object or null (no saved state)
```

### ZipEmbed.clearState()
Clear all saved state (localStorage, undo/redo stacks, and server). Used for "Start Over" / reset functionality.

```javascript
await ZipEmbed.clearState();
```

### ZipEmbed.undo() / ZipEmbed.redo()
In-memory only, per session. Max 50 entries.

```javascript
const prev = ZipEmbed.undo();   // Previous state or null
const next = ZipEmbed.redo();   // Next state or null
```

### ZipEmbed.hasUnsyncedChanges()
Returns `true` if localStorage state differs from server.

### ZipEmbed.onSyncStatus(callback)
Fires with sync status changes. Use this to show a save indicator.

```javascript
ZipEmbed.onSyncStatus(function(status) {
  // status: 'synced' | 'dirty' | 'syncing' | 'error'
  var labels = {
    synced: 'Saved',
    dirty: 'Unsaved changes',
    syncing: 'Saving...',
    error: 'Save failed'
  };
  document.getElementById('save-indicator').textContent = labels[status];
});
```

### ZipEmbed.syncNow()
Force immediate server sync instead of waiting for the 60-second timer.

```javascript
await ZipEmbed.syncNow();
```

---

## State Persistence Internals

- **Sync timer**: 60-second interval syncs dirty state to server via the embed wrapper
- **MD5 hash**: Server compares hash to skip redundant writes
- **beforeunload**: On tab close, bridge sends state to wrapper which fires `navigator.sendBeacon`
- **localStorage key**: `zipembed-state-{contentId}-{assignmentId}-{userId}`
- **Size limit**: 10 MB per state
- **Undo/redo**: In-memory only, max 50 entries, cleared on page reload

---

## Error Handling Pattern

Always handle submission failures with retry capability:

```javascript
ZipEmbed.submitScore(score, { artifacts: data })
  .then(function(result) {
    if (result.success) {
      submitBtn.textContent = 'Submitted!';
      submitBtn.disabled = true;
    } else {
      submitBtn.textContent = 'Retry Submit';
      submitBtn.disabled = false;
    }
  })
  .catch(function(err) {
    submitBtn.textContent = 'Retry Submit';
    submitBtn.disabled = false;
  });
```
