# HTML Playground

A lightweight, secure, single-file HTML playground hosted via GitHub Pages.

Split editor and live preview environment similar to online HTML compilers — fully self-contained (no third-party requests, not even web fonts), open source, and sandboxed for safety. Mobile-first design with full-screen Code/Preview toggle on phones and tablets.

**[Live Demo](https://adrianspeyer.github.io/html-playground/)**

---

## Features

- Split-pane layout (editor + preview) with draggable divider
- Mobile-first full-screen mode — toggle between Code and Preview via bottom bar
- HTML / CSS / JS tabs
- Manual **Run** button with optional **Auto-run**
- Console panel with collapse/expand, copy, and clear
- Light / Dark theme toggle (persisted across sessions)
- External CDN libraries supported
- LocalStorage persistence for code and preferences
- Hard **Reset Preview** to kill runaway scripts
- **Export** — download your merged HTML/CSS/JS as a standalone `.html` file
- **Import** — open an `.html` file from your device (Files / iCloud Drive on iPad) and view it whole in the sandbox; the Code view shows the full source
- **Fullscreen preview** — pop the preview to true OS-level fullscreen on iPad and desktop (falls back to full-screen Preview view on iPhone)
- Keyboard accessible — tabs, console, and pane divider all operable without a mouse
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
| `base-uri` | `'none'` | Block `<base>` tag hijacking of relative URLs |
| `form-action` | `'none'` | Block all form submissions |
| `script-src` | `'unsafe-inline' https:` | Allow inline scripts and HTTPS CDN scripts |
| `style-src` | `'unsafe-inline' https:` | Allow inline styles and HTTPS CDN stylesheets |
| `img-src` | `data: blob: https:` | Allow images from data URIs, blobs, and HTTPS |
| `font-src` | `data: https:` | Allow fonts from data URIs and HTTPS |
| `connect-src` | `https:` | Allow fetch/XHR to HTTPS endpoints |
| `media-src` | `blob: https:` | Allow audio/video from blobs and HTTPS |
| `frame-src` | `https:` | Allow the preview to embed HTTPS iframes (e.g. YouTube) |

> Note: `frame-ancestors` is intentionally **not** included — the CSP is delivered via a `<meta>` tag, and per spec, `frame-ancestors` is ignored in meta-delivered policies. Protection against the preview being framed comes from the iframe `sandbox` attribute instead. (`frame-src`, by contrast, controls what the preview itself is allowed to embed, and is permissive by design so demos can pull in HTTPS embeds.)

**Console bridge security:** The `postMessage` listener verifies that incoming messages originate from the preview iframe `contentWindow` before processing, preventing spoofed console messages from other sources.

**Hosting & shared origin (GitHub Pages):** All `*.github.io` user sites share a single origin, so this page's `localStorage`/cookies live in the same bucket as every other project on the account. The preview is protected from that shared storage because its iframe is sandboxed with `allow-scripts` **only** — no `allow-same-origin` — giving it a null origin that cannot read the host origin's storage. **This is the one invariant never to break:** adding `allow-same-origin` would let preview code read and wipe the `localStorage` and cookies of every project on the domain. The `allow-scripts`-only sandbox also blocks top-level navigation (no redirecting the visitor's tab), popups, and form submission. GitHub Pages can't set HTTP security headers, so the page can be framed by others; for an unauthenticated, stateless playground the clickjacking impact is negligible, and no host-page injection surface exists (no URL params are read).

### Known Tradeoffs

- **External scripts are allowed.** Because `script-src` permits any HTTPS source, user code can load and execute arbitrary remote JavaScript. This is by design for a personal/educational tool. If you are adapting this for a multi-tenant platform, restrict `script-src` to specific CDN domains.
- **`connect-src https:` is broad.** Preview code can make network requests to any HTTPS endpoint. This enables APIs and fetch-based workflows but means untrusted code could exfiltrate data visible within the iframe sandbox.
- **HTML merging uses regex.** The function that injects CSP and the console bridge into user HTML uses regular expressions to locate head, body, etc. Rare edge cases (e.g., those strings inside HTML comments or attribute values) could cause misinjection. Low risk for typical usage. Note: all replacements containing user content use function replacers (so `$&`-style replacement patterns in user code are never interpreted), and `</script>` / `</style>` sequences in user code are escaped to prevent breaking out of the injected wrapper tags.

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

### Import & Fullscreen

**Import** reads a single `.html` file with `FileReader` and places its full text into the HTML tab, clearing the CSS and JS tabs (an imported document already carries its own `<style>`/`<script>`). It then runs through the normal merge path, so the playground's CSP and console bridge still wrap the imported document — useful for safely opening and interacting with an HTML file on an iPad. Because the same CSP applies, an imported page that pulls resources over plain `http:` or relies on directives the policy denies may render differently than it would standalone; use **Export** for an unwrapped copy.

**Fullscreen** calls `requestFullscreen()` on the preview pane. On iPhone Safari (no element-fullscreen) it falls back to the full-screen Preview view. The Preview header — including the exit control — stays visible inside fullscreen.

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

The entire application is a single `index.html` file plus this README — no build step, no backend, and no third-party requests at runtime. The fonts use a native stack (San Francisco / SF Mono on Apple devices, system UI elsewhere), with `DM Sans` / `JetBrains Mono` kept first so they're used if installed. Single file by design.

---

## Changelog

### v1.6.0

**iPad / touch development**

- **Fixed silent code corruption on iOS** — the editor now sets `autocorrect`, `autocapitalize`, and `autocomplete` to `off`. Previously iOS smart-punctuation could turn a straight `"` into a curly `"` and autocapitalize line starts, silently breaking code.
- **Coding-key accessory bar** — on touch devices, a horizontally-scrollable bar of code keys (Tab, `( )`, `{ }`, `[ ]`, `< >`, `=`, `;`, `:`, `/`, `.`, quotes, `-`, `_`, `$`) appears above the on-screen keyboard. Bracket and quote keys insert as pairs with the caret placed between them. It rides above the keyboard via the VisualViewport API and stays hidden when a hardware keyboard is attached.
- **Fixed editor toolbar crowding on phones** — the file-tab strip now clips and scrolls instead of letting tab labels spill over the action buttons on narrow screens; mobile spacing tightened so all three tabs fit on a normal phone.

**Hardening**

- **Self-hosted typography** — removed the Google Fonts hotlink. No third-party requests at runtime, no visitor-IP leak to Google, and the "self-contained" claim is now literally true. Falls back to native system fonts (which on iPad/Mac are San Francisco and SF Mono).
- **Visible save failures** — `localStorage` write failures (iOS Private Browsing, storage pressure) now surface a one-time console warning pointing to Export, instead of being swallowed silently.
- **Import size guard** — files above 15 MB are refused before reading, so a huge file can't freeze the editor.
- **`allow-same-origin` guard comments** — both sandbox assignments carry an explicit warning never to add `allow-same-origin`, since on a shared `github.io` origin it would expose every project's storage on the domain.

### v1.5.0

**New**

- **Import HTML** — open an `.html` / `.htm` file from your device via the **Import** button (desktop header) or the mobile **More** menu. The file loads whole into the HTML tab (CSS/JS tabs are cleared, since an imported document is self-contained), runs immediately inside the sandbox, and on phones / iPad-portrait jumps straight to the full-screen preview. The Code view then shows the full imported source. Importing over unsaved work (anything other than the starter template) asks for confirmation first.
- **Fullscreen preview** — a fullscreen toggle in the Preview header uses the Fullscreen API for true OS-level fullscreen on iPad and desktop. iPhone Safari has no element-fullscreen, so it falls back to the existing full-screen Preview view. This covers iPad **landscape** (1024px), where the mobile full-screen toggle does not apply.

**Fixes / internal**

- **Docs ↔ code CSP sync** — the README Security Model table now matches the policy actually emitted by `buildCsp()`, including the previously-undocumented `base-uri 'none'`, `font-src`, `media-src`, and `frame-src` directives, with clarified `frame-src` vs `frame-ancestors` wording.
- **Generalized confirm dialog** — the confirmation modal is now reusable (title/body/button text are set per action) and drives both New Project and Import-over-unsaved-work, instead of being hard-wired to a single action.

### v1.4.0

**Fixes**

- **Replacement-pattern bug** — user CSS/JS containing `$&`, `` $` ``, or `$'` was being mangled during merge because it was passed as a `String.replace()` replacement string; all merge replacements now use function replacers
- **`</script>` / `</style>` breakout** — user code containing these literal sequences could terminate the injected wrapper tags and break the preview; they are now escaped (`<\/script`, `<\/style` — semantically identical inside JS and CSS strings)
- **Tab key preserved undo** — Tab indentation now uses `insertText` instead of reassigning `textarea.value`, so Ctrl/Cmd+Z works after indenting
- **CSP accuracy** — removed the no-op `frame-ancestors` directive (ignored in meta-delivered CSP per spec; framing protection comes from the iframe sandbox)

**New**

- **Export** — download the merged document (without the playground's CSP/console bridge) as a standalone `playground.html`; available in the desktop header and the mobile More menu
- **Keyboard accessibility** — editor tabs are a proper ARIA tablist (Enter/Space to activate, arrow keys to move), the console header toggles with Enter/Space and reports `aria-expanded`, and the pane divider is a focusable ARIA separator resizable with arrow keys; all three have visible focus indicators

### v1.3.0

- **Mobile full-screen mode** — on screens ≤820px, editor and preview are full-screen views toggled via a segmented control in a fixed bottom bar (replaces the stacked split layout)
- **Mobile bottom bar** — Code/Preview toggle, Run button, console toggle with log count badge, and a More menu for Auto-run, Reset Preview, Reset to Starter, and New Project
- **Run + switch** — tapping Run on mobile automatically switches to full-screen preview
- **Preview on tap** — switching to Preview auto-runs the code so the output is always fresh
- **Safe area support** — bottom bar respects `env(safe-area-inset-bottom)` for devices with home indicators (iPhone X and later)
- **Viewport lock** — `maximum-scale=1, user-scalable=no` prevents iOS double-tap zoom on the textarea
- **Larger mobile editor font** — textarea bumped to 14px on mobile for readability
- **Console badge** — log count badge on the mobile console button shows how many messages have been captured
- **Mobile More menu** — overlay menu provides access to all secondary actions (Auto-run, Reset Preview, Reset to Starter, New Project)
- **Synced autorun** — desktop and mobile Auto-run checkboxes stay in sync
- **Desktop unchanged** — side-by-side layout, draggable divider, and all header controls remain identical

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
