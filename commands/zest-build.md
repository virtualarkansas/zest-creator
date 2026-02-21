---
description: Validate and package Zest interactive content into a zip file ready for upload. Checks for required files, bridge script inclusion, and Canvas compatibility issues.
---

# /zest-build — Validate and Package Content

Package the current content directory into a zip file ready for upload to Zest.

## Process

### 1. Find the Content

Look for `index.html` in the current directory or the most recently created content subdirectory. If there are multiple content directories, ask the teacher which one to build.

### 2. Validation Checks

Run these checks and report results:

#### Required Files
- [ ] `index.html` exists
- [ ] If content uses `submitScore()` or `submitWork()`: `review.html` exists
- [ ] `zest.json` exists (recommended — auto-configures content on upload)

#### Spec File (`zest.json`)
- [ ] Valid JSON syntax
- [ ] `grading` field matches content behavior (`auto`, `teacher`, or `none`)
- [ ] `mainFile` matches actual entry file (default: `index.html`)
- [ ] If `reviewFile` declared: file exists in the content directory
- [ ] If `answerKey` declared: file exists and contains valid JSON
- [ ] If `parameters` declared: each parameter has `type`, `label`, and `default`
- [ ] If `sandbox` declared: no dangerous permissions without justification (`allowTopNavigation: true` is a red flag)

#### Answer Key
- [ ] If `answerKey` declared in `zest.json`: file exists and is valid JSON
- [ ] Answer key data is NOT hardcoded in `index.html` (security check)
- [ ] `review.html` uses `Zest.getAnswerKey()` instead of hardcoded values

#### Bridge Script
- [ ] `index.html` includes `<script src="https://lti.testyturtle.dev/public/zest-bridge.js"></script>`
- [ ] `review.html` (if present) includes the bridge script

#### Canvas Compatibility
- [ ] No `confirm()` calls (blocked in cross-origin iframes)
- [ ] No `alert()` calls (blocked in cross-origin iframes)
- [ ] No `prompt()` calls (blocked in cross-origin iframes)

#### Content Quality
- [ ] All JS wrapped in IIFE or module pattern
- [ ] `Zest.onReady()` is used as the entry point
- [ ] If using state persistence: `gatherState()` and `restoreState()` functions exist
- [ ] If using grading: error handling with retry on submission failure
- [ ] If using parameters: `Zest.getParameter()` used instead of hardcoded values

### 3. Report Issues

If any checks fail, report them clearly:

**Errors** (must fix before packaging):
- Missing `index.html`
- Missing `review.html` when grading is used
- Missing bridge script include

**Warnings** (should fix but won't block packaging):
- `confirm()`/`alert()`/`prompt()` usage — suggest two-click pattern
- Missing error handling on submit
- Missing state persistence for long-form content

**Notes** (informational):
- External CDN resources detected (list them)
- Grading mode detected (auto-graded / teacher-graded / none)
- State persistence detected (yes / no)

### 4. Package

If there are no blocking errors, create the `.zest` file (a zip with the `.zest` extension):

```bash
cd <content-directory>
zip -r ../<directory-name>.zest . -x ".*" -x "__MACOSX/*"
```

Exclude hidden files and macOS metadata. The `.zest` extension is a renamed zip — the server accepts both `.zip` and `.zest` files.

### 5. Summary

Report:
- **Zest file**: path and size
- **Files included**: count and list
- **Grading mode**: auto-graded / teacher-graded / none
- **State persistence**: yes / no
- **External resources**: list of CDN URLs (if any)
- **Validation**: PASS / PASS WITH WARNINGS / FAIL

### 6. Next Steps

Tell the teacher:
1. Download the `.zest` file
2. In Canvas, open the Rich Content Editor
3. Click the Zest toolbar button ("Embed Interactive Content")
4. Upload the `.zest` file
5. If the content has parameters, configure them in the picker UI
6. Choose "Interactive" embed mode (for graded content) or "Static" (for non-graded)
7. The content will appear as an iframe in the Canvas page
