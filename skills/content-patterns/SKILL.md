---
name: content-patterns
description: ZipEmbed content structure patterns and conventions. Use when creating HTML/JS/CSS interactive content for Canvas, building quizzes, labs, simulations, or any embeddable activity. Covers file structure, state persistence patterns, grading integration, and Canvas iframe constraints.
user-invocable: false
---

# ZipEmbed Content Patterns

## File Structure

Every ZipEmbed interactive has this structure:

```
my-content/
  index.html       # Student view (REQUIRED)
  review.html      # Teacher SpeedGrader view (required if graded)
```

Content is self-contained — all CSS and JS are inline in the HTML file. The only external dependency is the bridge script.

## HTML Template

Every index.html follows this skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Activity Title</title>
  <script src="https://lti.testyturtle.dev/public/zipembed-bridge.js"></script>
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
      ZipEmbed.onReady(function(ctx) {
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
    // Collect all saveable UI state
    answers: { q1: getSelectedValue('q1'), q2: getSelectedValue('q2') },
    currentPhase: currentPhase,
    submitted: isSubmitted
  };
}

function restoreState(state) {
  if (!state) return false;
  // Rebuild UI from saved state
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
    ZipEmbed.saveState(gatherState());
  });
});

// Debounced save for text inputs (1 second delay)
var saveTimeout = null;
document.querySelectorAll('textarea, input[type="text"]').forEach(function(el) {
  el.addEventListener('input', function() {
    if (saveTimeout) clearTimeout(saveTimeout);
    saveTimeout = setTimeout(function() {
      ZipEmbed.saveState(gatherState());
    }, 1000);
  });
});
```

### 3. Load and Restore in onReady

```javascript
ZipEmbed.onReady(function(ctx) {
  // Show user info, sync indicator, etc.

  ZipEmbed.loadState().then(function(state) {
    if (state) {
      var restored = restoreState(state);
      if (restored) {
        // Show "Your previous progress has been restored" banner
        document.getElementById('restored-banner').classList.add('visible');
      }
    }
  });
});
```

### 4. Sync Status Indicator

Always show a sync indicator so students know their work is saved:

```html
<div class="sync-indicator" id="sync-status" style="display:none;">
  <span class="sync-dot synced" id="sync-dot"></span>
  <span id="sync-text">Saved</span>
</div>
```

```css
.sync-indicator { font-size: 12px; color: #888; display: flex; align-items: center; gap: 6px; }
.sync-dot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; }
.sync-dot.synced { background: #27ae60; }
.sync-dot.dirty { background: #f39c12; }
.sync-dot.syncing { background: #3498db; animation: pulse 1s infinite; }
.sync-dot.error { background: #e74c3c; }
@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
```

```javascript
ZipEmbed.onSyncStatus(function(status) {
  document.getElementById('sync-dot').className = 'sync-dot ' + status;
  var labels = { synced: 'Saved', dirty: 'Unsaved changes', syncing: 'Saving...', error: 'Save failed' };
  document.getElementById('sync-text').textContent = labels[status] || status;
});
```

### 5. Restored Banner

```html
<div id="restored-banner" class="restored-banner">
  Your previous progress has been restored.
</div>
```

```css
.restored-banner {
  background: #d1ecf1; color: #0c5460; border: 1px solid #bee5eb;
  border-radius: 8px; padding: 10px 16px; font-size: 13px;
  margin-bottom: 16px; display: none; align-items: center; gap: 8px;
}
.restored-banner.visible { display: flex; }
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
    // Auto-cancel after 4 seconds
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

  // Perform the actual reset
  // 1. Clear all form fields
  // 2. Reset UI state
  // 3. Clear saved state
  ZipEmbed.clearState();
}
```

## SVG Chart Pattern

For data visualization, use inline SVG with viewBox for responsive scaling:

```javascript
var W = 380, H = 160;
var pad = { top: 12, right: 16, bottom: 28, left: 36 };
var plotW = W - pad.left - pad.right;
var plotH = H - pad.top - pad.bottom;

function xPos(i, total) { return pad.left + (i / (total - 1)) * plotW; }
function yPos(val, max) { return pad.top + plotH - (val / max) * plotH; }

var svg = '<svg viewBox="0 0 ' + W + ' ' + H + '" xmlns="http://www.w3.org/2000/svg">';
// Add grid lines, axes, paths, dots...
svg += '</svg>';
container.innerHTML = svg;
```

CSS for responsive SVG:
```css
.chart-container { position: relative; width: 100%; }
.chart-container svg { width: 100%; height: auto; display: block; }
```

## Button Styling

```css
.btn {
  display: inline-block; padding: 10px 24px; border: none; border-radius: 8px;
  font-size: 14px; font-weight: 600; cursor: pointer; transition: all 0.2s;
}
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-primary { background: #0770A2; color: white; }  /* auto-graded */
.btn-primary:hover:not(:disabled) { background: #065a82; }
.btn-submit { background: #2e7d32; color: white; }    /* teacher-graded */
.btn-submit:hover:not(:disabled) { background: #1b5e20; }
.btn-reset {
  background: transparent; color: #c0392b; border: 1px solid #e0e0e0;
  padding: 10px 20px; border-radius: 8px; font-size: 13px; cursor: pointer;
}
.btn-reset:hover { background: #fdf0ef; border-color: #c0392b; }
```

## Example Content

For detailed working examples of these patterns:
- [Auto-graded quiz pattern](auto-graded-example.md) — radio buttons, auto-scoring, submitScore
- [Teacher-graded lab pattern](teacher-graded-example.md) — multi-phase, textarea inputs, submitWork, simulation
