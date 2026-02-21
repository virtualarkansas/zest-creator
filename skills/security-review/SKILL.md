---
name: security-review
description: Review Zest interactive content for security vulnerabilities and data exfiltration risks. Use when an IT admin asks to review, audit, or check the security of uploaded Zest content, HTML embeds, or interactive activities.
---

# Zest Security Review

IT admins use this skill to review interactive content before deployment. Content runs inside Canvas iframes with sandbox permissions, so it has significant capabilities.

## Security Checklist

### 1. External Network Calls (Critical)
- `fetch(`, `XMLHttpRequest`, `navigator.sendBeacon(`, `new WebSocket(`, `new EventSource(`
- `<img src=` / `new Image()` — tracking pixels
- `<script src=` — external scripts beyond the bridge
- Legitimate: `lti.testyturtle.dev`, CDNs, `data:` URLs

### 2. Data Exposure (High)
- Direct `localStorage`/`sessionStorage` calls outside the bridge
- `document.cookie`, non-Zest `postMessage` calls

### 3. Code Execution (Medium)
- `eval(`, `new Function(`, `document.write(`, dynamic script injection

### 4. Browser API Abuse (Low)
- `navigator.geolocation`, `navigator.mediaDevices`, fingerprinting APIs

### 5. Answer Key Exposure (Critical)
- Answer keys should NOT be hardcoded in `index.html`
- Should be declared in `zest.json` and accessed via `Zest.getAnswerKey()` in review mode only

### 6. Sandbox Configuration (Medium)
- `allowModals: true` is unusual
- `allowTopNavigation: true` is dangerous

For thorough scanning, use the `zest:security-reviewer` agent.
