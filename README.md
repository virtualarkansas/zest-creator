# ZipEmbed Content Creator

Create interactive Canvas LMS content using Claude Code. Build quizzes, labs, simulations, and other activities that embed directly in Canvas pages with grading and state persistence.

## What is ZipEmbed?

[ZipEmbed](https://github.com/virtualarkansas/zipembed) is a self-hosted Canvas LTI 1.3 tool that lets you embed interactive HTML/JS/CSS content in Canvas pages. This template repo provides a Claude Code plugin that helps you create that content conversationally.

## Getting Started

### 1. Create Your Workspace

Click **"Use this template"** on GitHub to create your own copy of this repo.

### 2. Open in Claude Code

Open your new repo in [Claude Code](https://claude.ai/claude-code) (CLI or Web).

### 3. Create Content

Tell Claude what you want to build:

> "Make me a 10-question multiple choice quiz about the American Revolution for my 8th grade history class"

Or use the slash command:

> `/zipembed-new`

Claude will ask you about grading, state persistence, and other options, then build the content.

### 4. Package for Upload

When your content is ready:

> `/zipembed-build`

This validates your content and packages it as a zip file.

### 5. Upload to Canvas

1. In Canvas, open the Rich Content Editor (edit any page or assignment)
2. Click the ZipEmbed toolbar button ("Embed Interactive Content")
3. Upload your zip file
4. Choose **Interactive** mode (for graded content) or **Static** (for non-graded)
5. Save — your content appears as an interactive iframe in the Canvas page

## Features

- **Auto-Graded Content** — Quizzes, fill-in-the-blank, matching exercises with scores that appear instantly in the Canvas gradebook
- **Teacher-Graded Content** — Lab reports, essays, experiments where the teacher reviews and assigns grades in SpeedGrader
- **State Persistence** — Students can close the tab and return later; their work is automatically saved
- **SpeedGrader Integration** — Custom review view shows student work to teachers
- **Security Review** — IT admins can scan content for security issues before deployment

## Plugin Commands

| Command | Description |
|---------|-------------|
| `/zipembed-new` | Create new interactive content through conversation |
| `/zipembed-build` | Validate and package content as a zip for upload |

## Requirements

- A ZipEmbed instance deployed and connected to your Canvas LMS
- Claude Code (CLI or Web)

## License

MIT
