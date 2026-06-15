# /inspect

Browser-level check of the running app using chrome-devtools MCP. Catches issues that code analysis and tests cannot: rendering problems, console errors, network failures, and performance regressions.

**Only applies when config `project_type` is `web`.** For non-web projects, skip this command entirely.

## Config

Read `.claude/workflow.config.json` for pages, viewports, lighthouse threshold, and smoke test instructions.

## Modes

`/inspect` runs in **light** mode by default. Use `/inspect full` for the complete suite.

| Mode | When to use | Tool calls |
|------|-------------|------------|
| **Light** (default) | After every implementation cycle | ~10-12 |
| **Full** | Before first PR, after UI-heavy changes, or when dev requests | ~25-30 |

## Prerequisites

- App must be running (use config `commands.start`)
- Chrome DevTools MCP must be connected

If the app is not running, start it first and wait for it to be ready.

---

## Light Mode (default)

### 1. Page load — changed pages + home

Identify which pages were affected by recent changes (check `git diff --name-only`). Always include the first page from config `inspect.pages`. Navigate to each affected page:
- Check for console errors (`list_console_messages`) — flag errors and warnings
- Check for failed network requests (`list_network_requests`) — flag 4xx/5xx
- Take a screenshot of the primary changed page only (not every page)

### 2. Console error triage

Categorize all errors found:
- **Blocking** — JavaScript errors, uncaught exceptions, framework errors
- **Warning** — deprecation notices, minor warnings
- **Ignorable** — third-party analytics, browser extension noise

### 3. Interactive smoke test

Follow the smoke test instructions from config `inspect.smoke_test`. Verify no console errors appeared during the interaction.

### Light Mode Output

Report page load results table, console errors (if any), and smoke test result. Verdict: **App looks good** or **N issues found**.

---

## Full Mode (`/inspect full`)

Runs everything in light mode, plus the following additional checks.

### 4. Full page load — all pages

Navigate to every page listed in config `inspect.pages` not already checked in light mode. For each: screenshot, console errors, network failures, verify content renders.

### 5. Lighthouse audit

Run a Lighthouse audit on the first page from config `inspect.pages`:
- Performance score
- Accessibility score
- Best Practices score
- Flag any score below config `inspect.lighthouse_threshold` as a concern

### 6. Performance trace

Run a performance trace on the main page:
- Start trace → interact with the page → stop trace
- Analyze for: long tasks (>50ms), layout shifts, slow network requests (>3s)

### 7. Responsive check

Test at viewport sizes from config `inspect.viewports` using `emulate` or `resize_page`.

For each viewport, navigate to the first two pages from config `inspect.pages`:
- Take a screenshot
- Check for horizontal overflow
- Check for overlapping elements or unreadable text
- Check that navigation is usable
- Check that interactive elements are tappable at mobile size

### Full Mode Output

All light mode output, plus Lighthouse scores, performance concerns, and responsive check table. Verdict: **App looks good** or **N issues found**.
