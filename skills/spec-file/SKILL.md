---
name: spec-file
description: Zest spec file (zest.json) schema and usage. Use when creating or editing a zest.json manifest for interactive content, configuring parameters, answer keys, sandbox permissions, or any metadata for a Zestable package.
user-invocable: false
---

# Zest Spec File (`zest.json`)

## Purpose

`zest.json` is an optional manifest at the root of a Zest content package (`.zest` file). When present, it auto-configures the content on upload — setting the title, grading mode, entry files, parameters, sandbox permissions, and answer key.

## Full Schema

```json
{
  "name": "Activity Title",
  "version": "1.0.0",
  "description": "Short description of the activity",
  "author": "Teacher Name",
  "grading": "auto",
  "mainFile": "index.html",
  "reviewFile": "review.html",
  "answerKey": "answer-key.json",
  "embedType": "interactive",
  "width": 800,
  "height": 600,
  "autoHeight": false,
  "permissions": ["fullscreen"],
  "tags": ["biology", "quiz"],
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
    },
    "showHints": {
      "type": "boolean",
      "label": "Show Hints",
      "default": true
    },
    "studentName": {
      "type": "text",
      "label": "Custom Label",
      "default": ""
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

## Fields Reference

### Metadata
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | — | Display title for the content |
| `version` | string | — | Semantic version |
| `description` | string | — | Short description |
| `author` | string | — | Creator name |

### Content Configuration
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `grading` | string | `"none"` | `"auto"`, `"teacher"`, or `"none"` |
| `mainFile` | string | `"index.html"` | Student-facing entry file |
| `reviewFile` | string | auto-detected | SpeedGrader review file |
| `answerKey` | string | — | Path to answer key JSON (stored in `.secure/`) |
| `embedType` | string | `"interactive"` | Embed mode hint |

### Layout
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `width` | number | — | Suggested width in pixels |
| `height` | number | — | Suggested height in pixels |
| `autoHeight` | boolean | — | Auto-resize based on content |

### Categorization
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `permissions` | array | — | Requested browser permissions |
| `tags` | array | — | Tags for organizing content |

### Parameters

Parameters allow teachers to configure the content differently per-placement. Defined in `zest.json`, values are set by teachers during embedding.

**Parameter types:**

| Type | Value | Bridge returns |
|------|-------|---------------|
| `select` | One of `options` array | string |
| `number` | Numeric value within `min`/`max` | number |
| `boolean` | `true` or `false` | boolean |
| `text` | Freeform string | string |

Each parameter must have:
- `type` — one of the types above
- `label` — human-readable label shown in picker UI
- `default` — value used if teacher doesn't set one

Optional fields:
- `options` — array of choices (required for `select` type)
- `min`, `max` — numeric bounds (for `number` type)

**Reading parameters in content:**
```javascript
var difficulty = Zest.getParameter('difficulty', 'medium');
var timeLimit = Zest.getParameter('timeLimit', 30);
var params = Zest.getParameters(); // all parameters as object
```

### Sandbox

Controls the iframe sandbox permissions for the content. The server builds the `sandbox` attribute from these flags.

| Flag | Default | Sandbox Token |
|------|---------|---------------|
| `allowScripts` | `true` | `allow-scripts` |
| `allowSameOrigin` | `true` | `allow-same-origin` |
| `allowPopups` | `true` | `allow-popups` |
| `allowForms` | `true` | `allow-forms` |
| `allowModals` | `false` | `allow-modals` |
| `allowTopNavigation` | `false` | `allow-top-navigation` |

Default sandbox (when no `sandbox` field): `allow-scripts allow-same-origin allow-popups allow-forms`

## Answer Key

The `answerKey` field points to a JSON file in the content zip. During upload:

1. The file is moved to a `.secure/` subdirectory
2. The `.secure/` directory is blocked from public access (returns 403)
3. Students cannot access the answer key
4. Teachers can access it via `Zest.getAnswerKey()` in `review.html` (SpeedGrader only)

**Example answer key file (`answer-key.json`):**
```json
{
  "q1": "b",
  "q2": "c",
  "q3": "a",
  "q4": "d"
}
```

## Minimal Examples

### Auto-graded quiz
```json
{
  "name": "Chapter 3 Quiz",
  "grading": "auto",
  "answerKey": "answer-key.json"
}
```

### Teacher-graded lab
```json
{
  "name": "Plant Growth Experiment",
  "grading": "teacher",
  "parameters": {
    "difficulty": {
      "type": "select",
      "label": "Difficulty",
      "options": ["basic", "advanced"],
      "default": "basic"
    }
  }
}
```

### Ungraded tutorial
```json
{
  "name": "Periodic Table Explorer",
  "grading": "none",
  "tags": ["chemistry", "reference"]
}
```
