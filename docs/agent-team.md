# Agent team

To build Mona's Project Pulse dashboard, I'm using a team of four custom agents defined under `.github/agents/`, orchestrated with the GitHub Copilot CLI running in a Codespace.

- **Orchestrator** — Model: Claude Opus 4.7 (copilot). Coordinates the Planner, Coder, and Designer agents: breaks the request into phases, assigns non-overlapping file scopes, decides what can run in parallel vs. sequentially, and reports the final outcome. Defined in `.github/agents/orchestrator.agent.md`.
- **Planner** — Model: Claude Opus 4.7 (copilot). Researches the repository and requirements, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code. Defined in `.github/agents/planner.agent.md`.
- **Coder** — Model: GPT-5.5 (copilot). Implements the dashboard logic and structure within its assigned file scope, including support files like `.vscode/launch.json` for running the app. Defined in `.github/agents/coder.agent.md`.
- **Designer** — Model: Gemini 3.1 Pro (copilot). Owns UI/UX, accessibility, and visual design for the dashboard — project cards, status badges, priority treatment, spacing, and responsive layout. Defined in `.github/agents/designer.agent.md`.

All agents avoid staging, committing, or pushing changes; git operations stay under my control through Copilot CLI prompts.
