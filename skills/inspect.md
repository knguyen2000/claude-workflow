# /inspect

Browser-level check of the running app via chrome-devtools MCP. Catches what code analysis and tests can't: rendering, console errors, network failures, perf regressions.

**Web projects only** (`project_type: web`) — skip entirely otherwise.

## Config

Read `.claude/workflow.config.json` for pages, viewports, lighthouse threshold, smoke test instructions.

## Modes

`/inspect` = **light** (default, ~10-12 calls, after every implementation cycle). `/inspect full` = complete suite (~25-30 calls, before first PR / after UI-heavy changes / on request).

## Prerequisites

App running (`commands.start`) and Chrome DevTools MCP connected. Not running → start and wait.

---

## Light Mode

1. **Page load** — pages affected by `git diff --name-only`, plus config's first page always. Check console errors/warnings (`list_console_messages`), failed requests 4xx/5xx (`list_network_requests`), screenshot the primary changed page only.
2. **Error triage** — Blocking (JS errors, uncaught exceptions, framework errors) / Warning (deprecations, minor) / Ignorable (analytics, extension noise).
3. **Smoke test** — follow `inspect.smoke_test`; verify no console errors during the interaction.

**Output:** page load table, console errors (if any), smoke test result. Verdict: **App looks good** or **N issues found**.

---

## Full Mode

Everything in light mode, plus:

4. **All pages** — remaining `inspect.pages`: screenshot, console errors, network failures, content renders.
5. **Lighthouse** — first page: Performance/Accessibility/Best Practices scores; flag below `inspect.lighthouse_threshold`.
6. **Performance trace** — start → interact → stop; flag long tasks (>50ms), layout shifts, slow requests (>3s).
7. **Responsive** — `inspect.viewports` via `emulate`/`resize_page`, first two pages: screenshot, horizontal overflow, overlapping/unreadable text, usable nav, tappable elements.

**Output:** all light-mode output + Lighthouse scores + perf concerns + responsive table. Verdict: **App looks good** or **N issues found**.
