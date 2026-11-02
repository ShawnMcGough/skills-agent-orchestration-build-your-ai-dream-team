# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate the custom agent team building Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Agent definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinates the specialists, turns the plan into phases, assigns file ownership, and checks the integrated result without implementing it directly. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository and dependencies, then produces an ordered implementation plan with file assignments, edge cases, and validation expectations. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements dashboard logic and fixes within assigned files, validates behavior, and creates runnable app support such as `.vscode/launch.json` when assigned. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Creates a polished, accessible, responsive dashboard with project cards, status badges, clear priorities, and consistent visual styling within assigned files. | `.github/agents/designer.agent.md` |

The Orchestrator starts with the Planner's plan, then delegates to the Coder and Designer. Work runs in parallel only when file scopes do not overlap and there are no data dependencies; otherwise, it runs sequentially. The learner controls staging, commits, and pushes through Copilot CLI prompts.
