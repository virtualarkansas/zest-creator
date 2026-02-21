---
name: review-viewer
description: How to build review.html for Zest SpeedGrader integration. Use when creating the teacher review view that displays student-submitted artifacts, scores, and work in Canvas SpeedGrader.
user-invocable: false
---

# Review Viewer Pattern (review.html)

## Purpose

When a teacher opens SpeedGrader to grade a student's submission, Canvas loads `review.html` from the content zip. This file displays the student's submitted data (artifacts) in a read-only, teacher-friendly format.

## File Convention

If the zip contains `review.html` at the same directory level as `index.html`, SpeedGrader automatically loads it instead of `index.html`.

## HTML Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Review — Activity Title</title>
  <script src="https://lti.testyturtle.dev/public/zest-bridge.js"></script>
</head>
<body>
  <script>
    (function() {
      'use strict';
      Zest.onReady(function(ctx) {
        if (!Zest.isReviewMode()) {
          document.body.innerHTML = '<p>This page is only available in SpeedGrader review mode.</p>';
          return;
        }
        var sub = Zest.getSubmission();
        if (!sub) {
          document.body.innerHTML = '<p>No submission data found.</p>';
          return;
        }
        renderReview(sub);
      });

      function renderReview(sub) {
        // Build your review display here using sub.artifacts
      }
    })();
  </script>
</body>
</html>
```

## Submission Object

```javascript
var sub = Zest.getSubmission();
// { score, maxScore, artifacts, comment, submittedAt, userId, userName }
```

## Answer Key in Review Mode

If the content declared an `answerKey` in `zest.json`, the answer key is available via `Zest.getAnswerKey()`.

```javascript
var key = Zest.getAnswerKey();  // null if no answer key declared
if (sub && key) {
  // Compare student answers against the key
}
```

The answer key is **never sent to students** — only available in review mode (SpeedGrader).

## Important Notes

1. **Read-only**: review.html should never have interactive inputs.
2. **Always check `isReviewMode()`**: Gate the entire review rendering behind this check.
3. **Handle missing data**: Artifacts may have unexpected shapes from older content versions.
4. **Escape HTML**: Always escape user-generated text to prevent XSS.
5. **Same bridge script**: Include the same `zest-bridge.js` script tag.
6. **No state persistence needed**: review.html reads from the submission object.
7. **Answer key**: Use `Zest.getAnswerKey()` instead of hardcoding answers.
