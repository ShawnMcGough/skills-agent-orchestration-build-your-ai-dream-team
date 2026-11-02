# Project Pulse Dashboard – Implementation Plan

## 1. Overview / Goal

Project Pulse is a lightweight, static dashboard for Mona's team. In one glance, contributors can see:

- which projects are active
- who owns each project
- each project's current status
- recent activity
- priority or risk level
- a short contributor-friendly summary

It must be a polished frontend with project cards, status badges, clear priority treatment and readable spacing, not a plain HTML page. It is built from three static files in `app/` and opened through the VS Code **Run Project Pulse Dashboard** launch configuration. That configuration serves `app/` and opens `index.html`, never a directory listing.

The Orchestrator coordinates the work. The Planner (this document) defines phases, dependencies and file ownership. The Designer owns visual, layout and accessibility decisions. The Coder owns data, rendering logic and launch tooling. The learner controls all git operations.

## 2. File Assignments

| File | Owner | Purpose | Must contain |
| --- | --- | --- | --- |
| `app/project-data.json` | Coder | Source of truth for project data | Strict JSON. Top-level `"projects"` array with 4–6 projects. Each project has `name`, `owner`, `status`, `recentActivity` and `priority`, plus an optional `summary` string. Use a fixed set of values: `status` is one of `"On Track"`, `"At Risk"`, `"Blocked"` or `"Complete"`; `priority` is one of `"High"`, `"Medium"` or `"Low"`. |
| `app/index.html` | Coder (implements); Designer (markup direction) | Page shell and the logic that loads and renders data | `<html lang="en">`, a viewport meta tag, `<title>Project Pulse</title>`, and an `<h1>` that reads "Project Pulse". Links `styles.css`. Has a `<main class="dashboard">` container with a card grid region. Inline `<script>` calls `fetch("project-data.json")` and renders one `<article class="project-card">` per project, showing `name`, `owner`, `status`, `recentActivity` and `priority` (and `summary` if present). Inserts text with `textContent`, not `innerHTML`. Shows a visible error state if the fetch or parse fails. |
| `app/styles.css` | Designer | Visual design, layout and responsiveness | `.dashboard` and `.project-card` selectors. Badge classes such as `.status-badge` with modifiers for each status, and `.priority-badge` with modifiers for each priority. Cards use `border-radius` and `box-shadow`. A responsive CSS grid (`repeat(auto-fit, minmax(…))`) that collapses to one column on narrow screens. AA-contrast colours, visible `:focus-visible` styles, readable typography and spacing. Optional `prefers-reduced-motion` handling. |
| `.vscode/launch.json` | Coder | Run and preview the dashboard locally | Strict JSON with no comments. `"version": "0.2.0"` and one configuration named exactly **"Run Project Pulse Dashboard"**. Type `node-terminal`, `"command": "python3 -m http.server 5500"`, `"cwd": "${workspaceFolder}/app"`. A `serverReadyAction` with a pattern matching the server's port line (e.g. `"Serving HTTP on .* port ([0-9]+)"`), `"uriFormat": "http://localhost:%s/index.html"` and `"action": "openExternally"`. |
| `docs/project-pulse-plan.md` | Planner | This plan | n/a |

## 3. Designer Responsibilities

- **Design direction for `app/index.html` markup** (delivered to the Coder as a class/structure contract, not as edits to the file):
  - header containing `<h1>Project Pulse</h1>` and a short subtitle
  - `<main class="dashboard">` → `<section class="project-grid" aria-label="Projects">` → `<article class="project-card">`
  - inside each card: `<h2 class="project-name">`, `.project-owner`, `.status-badge status-<slug>`, `.priority-badge priority-<slug>`, `.project-activity`, `.project-summary`
  - `.dashboard-error` and `.dashboard-empty` states
- **Information hierarchy:** the project name is most prominent; status and priority badges come next (top of the card); then owner, recent activity and summary.
- **Visual styling (`app/styles.css`):** colour tokens via CSS custom properties, rounded cards (`border-radius`), soft `box-shadow`, hover/focus lift, generous spacing and a system font stack.
- **Responsive layout:** grid of 1 column on mobile (≤600px), 2 on tablet and 3+ on desktop. No horizontal scrolling at 320px.
- **Accessibility:**
  - text and badges meet WCAG AA contrast
  - status and priority are never conveyed by colour alone; the badge shows its text label
  - semantic landmarks
  - visible focus indicator
  - honour `prefers-reduced-motion`
- Report design decisions and any tradeoffs to the Orchestrator.

## 4. Coder Responsibilities

- **`app/project-data.json`:** realistic sample data following the schema in Section 2, with at least one project for every status and priority value so all badge styles are exercised.
- **`app/index.html`:** structure and logic.
  - Use the agreed class names exactly.
  - Turn status and priority values into lowercase hyphenated class modifiers (`"At Risk"` → `status-at-risk`).
  - Handle edge cases:
    - fetch failure or non-OK HTTP response
    - invalid JSON
    - a missing or empty `projects` array (show the empty state)
    - missing fields (show a placeholder such as "—")
  - Log errors to the console only when they occur.
  - Use no external dependencies or CDNs.
- **`.vscode/launch.json`:** exactly as specified in Section 2. The `cwd` must be `${workspaceFolder}/app` so that `/index.html` resolves and the browser does not show a directory listing.
- Validate the work (Section 7) and report results and remaining risks.

## 5. Dependencies

| # | Task | Depends on | Reason |
| --- | --- | --- | --- |
| T1 | Data schema agreed (field names, allowed status/priority values) | n/a | Defined in this plan |
| T2 | Markup/class-name contract (Designer) | T1 | Badge modifiers come from the status and priority values |
| T3 | Write `app/project-data.json` (Coder) | T1 | Must follow the schema |
| T4 | Write `.vscode/launch.json` (Coder) | n/a | Independent of the app's content |
| T5 | Write `app/styles.css` (Designer) | T2 | Styles target the agreed class names |
| T6 | Write `app/index.html` structure and render logic (Coder) | T1, T2 (T3 to test) | Renders the schema using the agreed classes |
| T7 | Integration review and polish | T3–T6 | Requires all files |

## 6. Parallel Work Decisions (Phases)

Parallel work is allowed only when file scopes don't overlap and there are no unresolved data dependencies.

- **Phase 1 – Contracts (sequential, short):** T1 is fixed by this plan, then the Designer delivers T2 (class/markup contract) to the Orchestrator. This must come first because both CSS and HTML depend on it.
- **Phase 2 – Build (parallel):**
  - Designer: `app/styles.css` (T5)
  - Coder: `app/project-data.json` (T3), `app/index.html` (T6) and `.vscode/launch.json` (T4)

  This is safe to run in parallel because each file has exactly one owner, the files don't overlap, and both sides work from the frozen contracts from Phase 1. Within the Coder's track, write T3 before T6 so rendering can be tested against real data. T4 can happen at any point.
- **Phase 3 – Integration and validation (sequential):** run the dashboard and complete the Section 7 checks. Fixes go back to each file's owner: CSS issues to the Designer; data, logic or launch issues to the Coder. Any class-name change requires both sides to agree through the Orchestrator.
- **Phase 4 – Handoff:** the Orchestrator summarises the results. The learner stages, commits and pushes.

## 7. Validation Expectations

- **Valid JSON:**
  - `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed
  - the data has a top-level `projects` array, and every entry has `name`, `owner`, `status`, `recentActivity` and `priority`
- **Page loads via launch config:**
  - **Run and Debug → Run Project Pulse Dashboard** starts `python3 -m http.server 5500` in `app/`
  - `serverReadyAction` opens `http://localhost:5500/index.html`, which shows the dashboard, not a directory listing
- **Data renders:**
  - one `.project-card` per project
  - every card visibly shows name, owner, status, recent activity and priority
  - the page title and `<h1>` read "Project Pulse"
- **Styles applied:**
  - `styles.css` loads (no 404)
  - `.dashboard` and `.project-card` styles are visible
  - cards are rounded and have shadows
  - status and priority badges are visually distinct
- **Responsive layout:** check widths of 320px, 768px and 1280px. The grid reflows from 1 to 3+ columns with no horizontal scrolling.
- **Accessibility basics:**
  - `lang` attribute, a single `<h1>` and heading order h1 → h2
  - landmarks (`header`, `main`)
  - AA contrast on badges and body text
  - badge text labels rather than colour alone
  - visible keyboard focus
  - Lighthouse or axe accessibility check with no critical issues
- **No console errors** on a normal load. Error states:
  - temporarily renaming the data file shows the visible error message
  - an empty `projects` array shows the empty state
- **Repository text checks:**
  - `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`
  - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`
  - `.vscode/launch.json` contains `Run Project Pulse Dashboard` and `index.html`

## Edge Cases & Risks

- Opening `index.html` via `file://` makes `fetch` fail, so the dashboard must always be served over HTTP through the launch config.
- If the `serverReadyAction` pattern doesn't match the server's output, no browser opens. Use a pattern that matches Python's "Serving HTTP on … port 5500" line.
- Port 5500 may already be in use. Stop earlier preview servers before relaunching.
- In Codespaces, `localhost:5500` is reached through automatic port forwarding. Confirm the forwarded port opens `/index.html`.
- Unknown status or priority values should fall back to a neutral badge style.

## Open Questions

- Should `summary` be required or optional? This plan makes it optional but recommends including it, since the brief asks for a contributor-friendly summary.
- Is the exact set of status and priority labels acceptable, or does Mona's team use different terms?
