# HTML Playground

A lightweight, secure, single-file HTML playground hosted via GitHub Pages.

Split editor and live preview environment similar to online HTML compilers — fully self-contained, open source, and sandboxed for safety.

**[Live Demo](https://adrianspeyer.github.io/html-playground/)**

---

## Features

- Split-pane layout (editor + preview) with draggable divider
- HTML / CSS / JS tabs
- Manual **Run** button with optional **Auto-run**
- Console panel with collapse/expand, copy, and clear
- Light / Dark theme toggle (persisted across sessions)
- External CDN libraries supported
- LocalStorage persistence for code and preferences
- Hard **Reset Preview** to kill runaway scripts
- Single `index.html` file — no build step, no backend

---

## Getting Started

### Local

Open `index.html` in any modern browser. That is it.

### GitHub Pages

1. Create a repository (or fork this one)
2. Add `index.html`
3. Commit and push
4. Go to **Settings > Pages > Deploy from branch > main / root**
5. Visit your GitHub Pages URL

---

## Security Model

The preview runs inside a sandboxed iframe with restricted capabilities.

**What the sandbox blocks:**

- `allow-same-origin` is **not** granted — the preview cannot access the parent page DOM, cookies, or localStorage
- Form submissions are blocked
- Top-level navigation is blocked
- Popups are blocked

**Content Security Policy (CSP)** is injected into every preview document to control resource loading:

| Directive | Value | Purpose |
|-----------|-------|---------|
| `default-src` | `'none'` | Deny everything by default |
| `script-src` | `'unsafe-inline' https:` | Allow inline scripts and HTTPS CDN scripts |
| `style-src` | `'unsafe-inline' https:` | Allow inline styles and HTTPS CDN stylesheets |
| `img-src` | `data: blob: https:` | Allow images from data URIs, blobs, and HTTPS |
| `connect-src` | `https:` | Allow fetch/XHR to HTTPS endpoints |
| `form-action` | `'none'` | Block all form submissions |
| `frame-ancestors` | `'none'` | Prevent framing of preview content |

**Console bridge security:** The `postMessage` listener verifies that incoming messages originate from the preview iframe `contentWindow` before processing, preventing spoofed console messages from other sources.

### Known Tradeoffs

- **External scripts are allowed.** Because `script-src` permits any HTTPS source, user code can load and execute arbitrary remote JavaScript. This is by design for a personal/educational tool. If you are adapting this for a multi-tenant platform, restrict `script-src` to specific CDN domains.
- **`connect-src https:` is broad.** Preview code can make network requests to any HTTPS endpoint. This enables APIs and fetch-based workflows but means untrusted code could exfiltrate data visible within the iframe sandbox.
- **HTML merging uses regex.** The function that injects CSP and the console bridge into user HTML uses regular expressions to locate head, body, etc. Edge cases (e.g., those strings inside HTML comments or attribute values) could cause misinjection. Low risk for typical usage.

### Recommendations for External Libraries

When loading third-party scripts via CDN, consider using [Subresource Integrity (SRI)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) hashes to ensure the file has not been tampered with.

---

## How It Works

### Editor

Code is stored across three tabs (HTML, CSS, JS). All state — including code, active tab, divider position, theme, console state, and auto-run preference — is persisted to `localStorage`.

### Run Flow

When **Run** is clicked (or auto-triggered on edit):

1. HTML, CSS, and JS from the three tabs are merged into a single document
2. A Content Security Policy meta tag is injected into head
3. A console bridge script is injected into body (overrides console.log/warn/error to forward messages to the parent via postMessage)
4. The merged document is assigned to `iframe.srcdoc`

### Console Panel

The console captures log, warn, error, and unhandled promise rejections from the preview. Features include:

- **Collapse / Expand** — click the console header to toggle
- **Copy** — copies all console output as formatted text
- **Clear** — wipes the console log
- **Color-coded levels** — INFO, WARN, ERROR each have distinct styling

### Theme Toggle

Click the sun/moon toggle in the header to switch between light and dark themes. The preference is saved to `localStorage` and restored on next visit. Theming is implemented entirely through CSS custom properties.

---

## Reset Preview

If user code contains infinite loops, heavy CPU tasks, or blocking operations, click **Reset Preview**. This completely replaces the iframe element in the DOM, immediately terminating all execution inside it.

---

## Using External Libraries

Include CDN scripts directly in the HTML tab. They load inside the sandboxed preview and are subject to the CSP.

---

## Limitations

- No file system access
- No npm/package manager support
- No TypeScript compilation
- No backend execution
- Infinite loops cannot be gracefully interrupted (must reset iframe)
- Single-file architecture means no multi-file project support

This is intentionally a pure client-side tool.

---

## Project Structure

The entire application is a single `index.html` file plus this README. Single file by design.

---

## Changelog

### v1.2.0

- **Select All toast** — "Selected!" feedback when clicking the Select All button
- **iPad / tablet support** — editor and preview stack vertically in portrait orientation (≤820px)
- **Mobile layout** — compact header, hidden button labels, and smaller controls at ≤600px
- **Touch-friendly divider** — 12px hit target on touch devices, works both horizontal and vertical
- **Keyboard shortcuts** — Tab inserts 2 spaces, Ctrl/Cmd+Enter runs code, Ctrl/Cmd+S prevented
- **Orientation handling** — layout re-applies correctly on device rotation
- **Safari drag fix** — added `-webkit-user-select` for stable divider dragging
- **100dvh support** — proper viewport height on mobile browsers with address bars
- **Scrollable header** — header scrolls horizontally on narrow screens instead of overflowing

The search is pulling other projects. I have the full history from the session transcripts though. Here's the complete changelog based on all our sessions:
markdown## Changelog

### v1.2.0

- **Select All toast** — "Selected!" feedback when clicking the Select All button
- **iPad / tablet support** — editor and preview stack vertically in portrait orientation (≤820px)
- **Mobile layout** — compact header, hidden button labels, and smaller controls at ≤600px
- **Touch-friendly divider** — 12px hit target on touch devices, works both horizontal and vertical
- **Keyboard shortcuts** — Tab inserts 2 spaces, Ctrl/Cmd+Enter runs code, Ctrl/Cmd+S prevented
- **Orientation handling** — layout re-applies correctly on device rotation
- **Safari drag fix** — added `-webkit-user-select` for stable divider dragging
- **100dvh support** — proper viewport height on mobile browsers with address bars
- **Scrollable header** — header scrolls horizontally on narrow screens instead of overflowing

### v1.1.0

- **Light / dark theme** — toggle with animated sun/moon icon, persisted to localStorage
- **Console panel** — collapsed by default, expand to view logs, copy output with toast feedback
- **Copy button toast** — "Copied!" tooltip animation on editor copy
- **Console copy toast** — "Copied!" feedback with checkmark icon swap
- **New Project dialog** — confirmation modal with overlay click and Escape key to dismiss
- **Clear tab button** — clears the active tab's code
- **Reset code button** — resets all tabs to the starter template
- **Hard reset preview** — recreates iframe to kill runaway scripts
- **Footer** — displays version number

### v1.0.0

- **Initial release** — single-file HTML playground
- **Split-pane editor** — draggable divider between code editor and live preview
- **Three-tab editor** — HTML, CSS, and JS tabs with localStorage persistence
- **Live preview** — renders via `iframe.srcdoc` with merged HTML/CSS/JS
- **Auto-run mode** — optional 400ms debounced auto-execution on input
- **Console capture** — intercepts `console.log`, `warn`, `error` via `postMessage` bridge
- **Sandboxed iframe** — `allow-scripts` only, no `allow-same-origin`
- **Content Security Policy** — injected CSP restricts resource loading in preview
- **XSS protection** — `escapeHtml()` sanitizes all console output including backticks
- **Strict mode** — all JavaScript runs under `"use strict"`
- **Source verification** — `postMessage` listener validates `ev.source` against iframe
- **Starter template** — default HTML/CSS/JS demo code on first load
---

## Contributing

Contributions are welcome. Please open an issue first to discuss any significant changes.

---

## License

MIT — use freely for personal, educational, or commercial purposes.
