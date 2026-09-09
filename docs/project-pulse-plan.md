# Project Pulse — Implementation Plan

## 1. Overview

Project Pulse is a lightweight, single-page project status dashboard. It is a fully static site: `app/index.html` loads `app/styles.css`, then fetches `app/project-data.json` at runtime and renders a grid of project cards in the browser. When a learner opens the site (via VS Code's Live Server or the provided `.vscode/launch.json` configuration), they see a titled dashboard header with a short subtitle, a summary strip (total projects, counts by status), and a responsive grid of project cards. Each card shows the project name, owner, status badge (e.g., On Track / At Risk / Blocked / Done), priority indicator, progress bar with percent complete, due date, and a short description. The page is keyboard-navigable, accessible, and responsive from ~360px mobile widths up to desktop.

## 2. File Assignments

| File | Owner Agent | Purpose | Key Contents / Sections |
|---|---|---|---|
| `app/index.html` | **Coder** | Semantic page skeleton and JS that fetches JSON and renders cards. | `<!doctype html>`, `<head>` with meta viewport + link to `styles.css`, `<body>` with `<header>` (title, subtitle), `<section id="summary">`, `<main id="dashboard">` container, inline `<script type="module">` (or `<script defer>`) that calls `fetch('./project-data.json')`, parses, and injects card markup; error/empty states; sets `aria-live` region for load status. |
| `app/styles.css` | **Designer** | Visual system for the dashboard: layout, palette, typography, components. | CSS custom properties (`:root` tokens for colors, spacing, radii, font stack), base/reset styles, header styles, summary strip, responsive CSS Grid for cards (`repeat(auto-fill, minmax(280px, 1fr))`), `.card`, `.badge--on-track/at-risk/blocked/done`, `.priority--high/med/low`, `.progress` bar, focus-visible outlines, prefers-reduced-motion, mobile/tablet/desktop breakpoints. |
| `app/project-data.json` | **Coder** (schema); **Designer** consumes for styling states | Mock data source rendered by the dashboard. | Top-level `{ "generatedAt": ISO-string, "projects": [ ... ] }`. Each project: `id` (string), `name`, `owner`, `status` (`"on-track"\|"at-risk"\|"blocked"\|"done"`), `priority` (`"high"\|"medium"\|"low"`), `progress` (0–100 integer), `dueDate` (ISO date), `description` (short string). Include ~6–8 sample projects covering every status and priority so the Designer can style all states. |
| `.vscode/launch.json` | **Coder** | One-click local preview for the learner from VS Code. | `version: "0.2.0"`, `configurations: [{ type: "chrome" \| "msedge", request: "launch", name: "Launch Project Pulse", file: "${workspaceFolder}/app/index.html" }]`. Document Live Server as the recommended alternative (because `fetch()` of a local JSON file will fail on `file://` in most browsers due to CORS). Prefer a `"url"` + `"webRoot"` config pointing at a Live Server URL, or add a Live Server usage note. |

## 3. Designer Responsibilities

**Files touched:** `app/styles.css` (owner). Read-only reference to `app/index.html` (for class hooks) and `app/project-data.json` (to ensure every status/priority value has a corresponding style).

**Direction:**
- **Layout:** Sticky-feeling header with product title "Project Pulse" and short subtitle. Below: a summary strip (flex row, wraps on mobile) showing total count and per-status counts. Below that: a responsive CSS Grid of cards using `repeat(auto-fill, minmax(280px, 1fr))` with ~1rem gap. Cards are vertical stacks: name → owner/due meta row → description → progress bar → footer row with status badge + priority indicator.
- **Palette (suggested tokens):**
  - Background: `#0f172a` dark or `#f8fafc` light (pick one primary theme; ship light by default).
  - Surface: `#ffffff` with subtle border `#e2e8f0` and soft shadow.
  - Text: `#0f172a` primary, `#475569` secondary.
  - Status: on-track `#16a34a`, at-risk `#d97706`, blocked `#dc2626`, done `#2563eb`.
  - Priority: high (solid red dot/label), medium (amber), low (slate).
  - Ensure all text/badge combinations meet WCAG AA (≥ 4.5:1 for body, ≥ 3:1 for large/badge text).
- **Typography:** System font stack (`-apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif`). Scale: title 1.75rem, card name 1.125rem, body 1rem, meta 0.875rem. Line-height 1.4–1.6.
- **Components:** `.card` (radius 12px, padding 1rem, shadow), `.badge` variants per status, `.priority` pill or dot, `.progress` (accessible bar using `<progress>` or a styled `<div role="progressbar" aria-valuenow>`), summary chips.
- **Accessibility:** Visible `:focus-visible` outlines, sufficient color contrast, non-color status cues (badge text label, not color alone), semantic landmarks respected, `prefers-reduced-motion` disables transitions, tap targets ≥ 40px.
- **Responsive:** Verify at 360px, 768px, 1280px. Cards go single-column under ~480px.

## 4. Coder Responsibilities

**Files touched:** `app/index.html` (owner), `app/project-data.json` (owner), `.vscode/launch.json` (owner).

**HTML structure (`app/index.html`):**
- Semantic landmarks: `<header>`, `<main>`, optional `<footer>`.
- Class hooks the Designer can style: `.summary`, `.summary__chip`, `.dashboard`, `.card`, `.card__header`, `.card__meta`, `.card__desc`, `.progress`, `.badge`, `.badge--{status}`, `.priority`, `.priority--{level}`.
- An `aria-live="polite"` status region for "Loading…", errors, and empty states.

**Data loading and rendering:**
- On `DOMContentLoaded`, `fetch('./project-data.json')`.
- Validate shape: top-level `projects` array exists; skip malformed entries and log a `console.warn`.
- Render summary counts (total + per-status).
- For each project, build a card via `document.createElement` (avoid `innerHTML` with untrusted data; use `textContent`).
- Format `dueDate` with `Intl.DateTimeFormat` and flag overdue dates (compare to today) with a subtle "Overdue" tag (still status-agnostic).
- Progress uses `<progress max="100" value="{progress}">` with `aria-label` including project name; also render numeric percent as text for redundancy.
- Handle empty array (friendly "No projects yet" message) and fetch failure (error message + retry hint).

**`app/project-data.json`:**
- Author the schema described in section 2. Include 6–8 entries covering every `status` and every `priority`, at least one overdue, and one with `progress: 100` + `status: "done"`.
- Ensure valid JSON (no trailing commas, no comments).

**`.vscode/launch.json`:**
- Provide a Chrome/Edge launch configuration. Because `fetch()` against `file://` typically fails, prefer configuring the launch to open a Live Server URL (e.g., `http://127.0.0.1:5500/app/index.html`) and document the Live Server extension as a prerequisite in a top-of-file comment or in the README. Include `webRoot: "${workspaceFolder}"`.
- Alternative acceptable: a task-backed compound that starts a static server, but keep it simple — Live Server note is fine for a learner project.

## 5. Dependencies

1. `app/project-data.json` schema must be defined **before** `app/index.html` rendering logic is finalized (rendering reads specific fields).
2. `app/styles.css` depends on the Designer's direction **and** on the class hooks/DOM structure declared in `app/index.html` (or an agreed contract of class names shared up front).
3. Full status/priority styling in `app/styles.css` depends on the enumerated values in `app/project-data.json` (so every state has a matching class).
4. `.vscode/launch.json` depends on `app/index.html` existing at the target path (and, for the Live Server URL variant, on the learner having the Live Server extension).
5. Validation (section 7) depends on all four files being present.

To decouple steps 1–2 and enable parallelism, publish a **shared contract early**: (a) the JSON schema in section 2 and (b) the class-name list in section 4. Both agents work against that contract.

## 6. Parallel vs. Sequential Work

**Sequential (must be done first):**
- **Step A — Contract definition (Planner/Coder, ~first):** Lock the JSON schema and the class-name list. Everything else depends on this.

**Parallel (after Step A):**
- **Step B (Coder):** Author `app/project-data.json` with sample data.
- **Step C (Coder):** Author `app/index.html` skeleton + fetch/render script against the agreed schema and class names.
- **Step D (Designer):** Author `app/styles.css` against the agreed class names and status/priority enums.

  Steps B, C, and D touch **non-overlapping files** and share no runtime state — safe to run concurrently.

**Sequential (after B–D):**
- **Step E (Coder):** Author `.vscode/launch.json`. Needs `app/index.html` to exist at its final path.
- **Step F (Integration check):** Open the dashboard, verify rendering with real styles, adjust any class-name mismatches. This is a serialization point where Designer + Coder reconcile.
- **Step G (Validation):** Run the checks in section 7.

**Justification:** File ownership is disjoint across B/C/D, and the shared contract removes the only cross-file coupling, so parallel execution is safe. E depends on C's output path. F/G require all prior outputs.

## 7. Validation Expectations

**Local preview:**
- **Preferred:** Install VS Code "Live Server" extension → right-click `app/index.html` → "Open with Live Server". This serves over `http://` so `fetch()` of `project-data.json` succeeds.
- **Alternative:** Use the `.vscode/launch.json` "Launch Project Pulse" configuration (which points at the Live Server URL).
- **Do not** double-click the file to open via `file://` — `fetch()` will be blocked in most browsers.

**Functional checks:**
- Page loads with no errors in the browser DevTools Console.
- Summary strip shows a total that matches the number of projects in the JSON, and per-status counts sum to the total.
- Every project in `project-data.json` appears as a card, with correct name, owner, due date (formatted), description, status badge, priority indicator, and progress bar reflecting the `progress` value.
- Overdue projects display an "Overdue" cue.
- Empty-state message renders if `projects: []`.
- Error-state message renders if `project-data.json` is missing or malformed (simulate by temporarily renaming the file).

**Visual / responsive checks:**
- Layout is intact at 360px, 768px, and 1280px viewport widths.
- Cards reflow to a single column on narrow screens; grid fills available width on wide screens.
- All status and priority variants have distinct, correctly-styled treatments.

**Accessibility checks (lightweight):**
- Keyboard: `Tab` moves focus through interactive elements; `:focus-visible` outline is clearly visible.
- Screen-reader spot-check: header is an `<h1>`, cards use semantic structure, progress bars expose an accessible name and current value.
- Color contrast for text and badges passes WCAG AA (verify with DevTools or axe DevTools extension).
- With `prefers-reduced-motion: reduce`, no non-essential animations play.

**Data / markup sanity:**
- `project-data.json` parses cleanly (`JSON.parse` in DevTools or `jq . app/project-data.json`).
- HTML validates (paste into the W3C Nu HTML Checker or use a VS Code HTML linter).
- No uncaught exceptions or 404s in the Network tab; `project-data.json` returns 200.

## Open Questions

1. **Theme:** Ship light-mode only, dark-mode only, or both via `prefers-color-scheme`? Plan assumes light-only for simplicity.
2. **Launch target:** Do we require the Live Server extension as a prerequisite, or should the Coder add a tiny static server script under `scripts/` and reference it from `launch.json`? Plan assumes Live Server.
3. **Interactivity scope:** Is filtering/sorting in scope for v1, or strictly read-only rendering? Plan assumes read-only.
4. **Data volume:** Fixed ~6–8 mock projects, or should the JSON be larger to stress-test the grid? Plan assumes 6–8.
