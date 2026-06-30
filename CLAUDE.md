# CLAUDE.md

## What this is

A single-file, **client-side** load tester for LLM chat-completion APIs. It fires N
parallel requests directly from the browser at an OpenAI-compatible or Anthropic
endpoint, measures overlap and (when streaming) time-to-first-token, and renders
a verdict (concurrent / partial / sequential) plus a timeline and per-request
table.

No build step, no server, no framework. Tailwind and the Inter font are loaded
from CDN. The user pastes their own API key; it is stored only in
`localStorage` (`llmct_*` keys) and sent directly to the endpoint they configure.

## Files

- `index.html` — the entire app. Holds HTML, inline CSS, and all JS.

That's it. There is no `package.json`, no source tree, no test harness.

## How to run it

Just open `index.html` in a browser. Any of these work:

- Double-click the file (loads over `file://`).
- `python3 -m http.server 8000` then visit `http://localhost:8000`.
- Drag the file onto a browser window.

CORS is the only constraint that matters: the API endpoint must allow
cross-origin requests from a browser origin, or every request fails with a
generic network error. There is no client-side fix for that — the page warns
about it explicitly.

## Code layout (inside `index.html`)

Top-to-bottom inside the single `<script>` block:

1. **Theme** — `toggleTheme()`, persists `llmct_theme`.
2. **Persistence** — `saveField` / `loadField` helpers around `localStorage`.
3. **Provider + endpoint normalization** — `currentProvider`, `normalizeEndpoint`,
   `setProvider`. Endpoint auto-resolves base URL → full path on blur.
4. **Run-button validation** — `validateRunButton()` gates the RUN TEST button.
5. **Init IIFE** — restores persisted fields, wires `input`/`blur` listeners.
6. **Presets** — `applyPreset('light' | 'heavy')` for quick configuration.
7. **Request execution** — `callOnce(idx, cfg, testStart)` does one request.
8. **Run orchestration** — `runTest()` dispatches N requests via `Promise.all`
   and renders results.
9. **Rendering** — `statBlock` helper and `render(data)` builds the stat boxes,
   verdict, timeline, and per-request table.

## Conventions

- **Tailwind for static layout, inline `style=` for anything computed at
  runtime.** The CDN tailwind build is JIT; dynamic class strings cause
  recompilation thrash. Dynamic widths, colors, and segment positions are set
  with `element.style.left = ...` etc., matching what the existing code does.
- **Neo-brutalist styling.** Custom CSS variables under `:root` (light) and
  `html.dark` (dark). Utility classes: `.neo-border`, `.neo-shadow`,
  `.neo-input`, `.neo-button`, `.neo-panel`, plus `.stat-box`, `.verdict-box`,
  `.track`, `.bar`, `.data-table`. Reuse these rather than introducing new
  visual primitives.
- **localStorage keys** are always prefixed `llmct_`.
- **Result-object shape** used by `callOnce` and consumed by `render`: see the
  object literal built at the top of `callOnce`. New fields should be added
  there and surfaced through `render` consistently.
- **Single-burst concurrency.** No rate/sustained-load mode. N requests fire as
  close to simultaneously as `Promise.all` allows; the verdict derives from
  `overlapRatio = sum(durations) / wallTime`.

## What this project does NOT do

- No server-side proxy. CORS failures are reported, not worked around.
- No streaming token-by-token display. The chart shows duration only.
- No TTFT measurement yet. (Tracked as a future feature — adding it requires
  switching `callOnce` to SSE, which is why a stream refactor would touch a
  lot at once.)
- No persistence of test history. Each run lives in the current DOM only.
- No automated tests. Verification is manual against real API endpoints.