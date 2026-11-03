# Project Pulse – Final Handoff

Project Pulse is a static dashboard for Mona's team. It shows each project's name, owner, status, recent activity, priority and summary as styled cards. It is built from three static files in `app/` and runs through a VS Code launch configuration.

## Agent team and contributions

| Agent | Model | Owned / contributed |
| --- | --- | --- |
| **Orchestrator** | Claude Opus 5.5 | Coordinated the team, assigned file scopes, reviewed every file against the plan, and ran the integration review. |
| **Planner** | Claude Opus 5.5 | Wrote `docs/project-pulse-plan.md`, which covers phases, dependencies, file ownership, the data schema and the validation expectations. |
| **Designer** | Gemini 3.7 Flash | Set the markup and class-name contract and wrote `app/styles.css`, which covers visual design, the responsive grid and accessibility. |
| **Coder** | GPT-5.5 | Wrote `app/project-data.json`, `app/index.html` (render logic) and `.vscode/launch.json`, ran validation and wrote this handoff. |

## Deliverables

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | Coder (markup direction from Designer) | Page shell with `lang="en"`, a viewport meta tag, and the title and `<h1>` "Project Pulse". It links `styles.css`. Inside `main.dashboard > section.project-grid`, an inline script calls `fetch("project-data.json")` and renders one `article.project-card` per project using `textContent`. It shows `.dashboard-error` (`role="alert"`) and `.dashboard-empty` states and uses `*-unknown` badge fallbacks. |
| `app/styles.css` | Designer | Design tokens, `.dashboard`, `.project-card` (`border-radius`, `box-shadow`) and status/priority badge modifiers. The grid uses `repeat(auto-fit, minmax(min(100%, 18rem), 1fr))`. Also includes `:focus-visible`, `prefers-reduced-motion` and `forced-colors` support. |
| `app/project-data.json` | Coder | Source of truth: a top-level `projects` array with 6 projects. Together they cover every status (On Track, At Risk, Blocked, Complete) and every priority (High, Medium, Low), and every project has a `summary`. |
| `.vscode/launch.json` | Coder | The **Run Project Pulse Dashboard** launch configuration. |

## How to run the dashboard

1. Open **Run and Debug** in VS Code (Ctrl/Cmd+Shift+D).
2. Select **Run Project Pulse Dashboard** (defined in `.vscode/launch.json`) and press ▶ / F5.
3. VS Code runs `python3 -m http.server 5500` in a terminal, with `cwd` set to `${workspaceFolder}/app`.
4. When the terminal prints `Serving HTTP on … port 5500`, `serverReadyAction` opens `http://localhost:5500/index.html`. This shows the dashboard, not a directory listing.
5. **In Codespaces:** port 5500 is auto-forwarded. If no browser tab opens, open the **Ports** panel, open the forwarded address for port 5500 and add `/index.html` to the URL.
6. To stop the server, press Ctrl+C in that terminal or stop the debug session.

## Dashboard validation results

Validation was run headlessly by the Coder. The test server used port 5599 so it wouldn't clash with 5500, and it was stopped afterwards. Temporary files were removed.

| # | Check (plan Section 7) | Command / method | Result |
| --- | --- | --- | --- |
| 1 | Data file is valid JSON | `python3 -m json.tool app/project-data.json` | ✅ Pass |
| 2 | Launch file is strict JSON | `python3 -m json.tool .vscode/launch.json` | ✅ Pass |
| 3 | Schema: top-level `projects` array, every entry has non-empty `name`, `owner`, `status`, `recentActivity`, `priority`; status/priority in allowed sets | Python `json.load` + assertions | ✅ Pass: 6 projects, all 4 statuses, all 3 priorities |
| 4 | `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | `grep -F` per string | ✅ Pass (7/7) |
| 5 | `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow` | `grep -F` per string | ✅ Pass (4/4) |
| 6 | `.vscode/launch.json` contains `Run Project Pulse Dashboard` and `index.html` | `grep -F` per string | ✅ Pass (2/2) |
| 7 | Files are served over HTTP | `cd app && python3 -m http.server 5599`, then `curl` each file | ✅ Pass: `/index.html` 200 `text/html`; `/styles.css` 200 `text/css`; `/project-data.json` 200 `application/json` |
| 8 | `/index.html` is the dashboard, not a directory listing | `curl` + grep | ✅ Pass: no "Directory listing" text; served `<html lang="en">`, `<title>Project Pulse</title>`, `<h1>Project Pulse</h1>`, `<main class="dashboard">` |
| 9 | Server startup line matches `serverReadyAction` pattern | Python `re.search(r"Serving HTTP on .* port ([0-9]+)", …)` on the real line `Serving HTTP on 0.0.0.0 port 5599 (http://0.0.0.0:5599/) ...` | ✅ Pass: captured port `5599` |
| 10 | Inline script syntax | Extracted the `<script>` block and ran `node --check` (Node v22.15.0) | ✅ Pass |
| 11 | Safe rendering / basics | `grep` in `app/index.html` | ✅ Pass: 0 `innerHTML`, exactly 1 `<h1>`, no external `http` script or style references |

**Defects found:** none.

### Manual checks for the learner

These checks need a real browser and can't be verified headlessly:

- [ ] Launch **Run Project Pulse Dashboard** and confirm the browser opens automatically at `/index.html`.
- [ ] Confirm the page shows 6 cards and that each shows name, owner, status, recent activity, priority and summary.
- [ ] Check that cards are rounded and shadowed and that status and priority badges are visually distinct.
- [ ] Check responsive widths at **320px, 768px and 1280px** in DevTools device mode. The layout should go from 1 column to 3+ with no horizontal scrolling.
- [ ] Press Tab through the page and confirm keyboard focus is visible.
- [ ] Run **Lighthouse** or **axe**. There should be no critical accessibility issues, and badges and body text should have AA contrast.
- [ ] Confirm the DevTools console has no errors on a normal load.
- [ ] Error states: temporarily rename `project-data.json` and confirm the visible error message appears. Temporarily set `"projects": []` and confirm the empty state appears. Revert both changes afterwards.

## Learner handoff and next steps

### Remaining risks

- **`file://` fetch failure:** opening `app/index.html` directly from disk blocks `fetch("project-data.json")`, so you'll see the error state instead of the cards. Always use the launch configuration, or any HTTP server rooted at `app/`.
- **Port 5500 in use:** if an earlier preview server is still running, the new one fails with "Address already in use" and no browser opens. Stop the old terminal or debug session and relaunch.
- **`serverReadyAction` pattern:** the browser only opens when the terminal output matches `Serving HTTP on .* port ([0-9]+)`. This matched Python's real output in validation. If a different Python version or a wrapper changes that line, open `http://localhost:5500/index.html` manually. In Codespaces, auto-open depends on port forwarding, so you can use the **Ports** panel instead.

### Open questions (from the plan)

- Should `summary` be required or optional? The plan makes it optional. All 6 sample projects include it.
- Are the status labels (On Track, At Risk, Blocked, Complete) and priority labels (High, Medium, Low) right for Mona's team, or does the team use different terms? Changing them means updating the data, the badge modifiers in `app/styles.css` and possibly the slug logic in `app/index.html`.

### Git

The learner controls all git operations. No agent has staged, committed or pushed anything. Review the changes, then stage, commit and push them yourself, for example through Copilot CLI prompts.
