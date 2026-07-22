# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent team defined under `.github/agents/` and orchestrated with GitHub Copilot CLI running in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases based on the Planner's file assignments and dependencies, delegates work with explicit file scopes, runs non-overlapping work in parallel, and reports the integrated result back to the learner. Does not implement anything itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository and relevant docs/dependencies, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable vs. sequential work, edge cases, and open questions. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code within the file scope assigned by the Orchestrator — building the Project Pulse dashboard logic and, when assigned, support configuration such as `.vscode/launch.json` (with `cwd` set to `${workspaceFolder}/app` and `index.html` opened by default). Validates changes before reporting completion.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design within the assigned scope. For Project Pulse, delivers a polished dashboard with project cards, status badges, priority treatment, and responsive layout, using deterministic CSS hooks like `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`

## Workflow note

All orchestration is done via GitHub Copilot CLI inside a Codespace — the Orchestrator delegates to Planner, Coder, and Designer as subagents, and I (the learner) retain control of all git operations (staging, committing, pushing).
