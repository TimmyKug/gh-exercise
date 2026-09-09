# Agent team

This team will use GitHub Copilot CLI in a Codespace to orchestrate the work of building Mona's Project Pulse dashboard.

## Custom agents

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | GPT-5.6 Luna (copilot) | Coordinates the specialist agents, breaks the work into phases, assigns non-overlapping file scopes, manages dependencies, verifies the integrated result, and reports outcomes. | `.github/agents/orchestrator.agent.md` |
| **Planner** | GPT-5.6 Luna (copilot) | Researches the repository, documentation, dependencies, edge cases, and risks, then produces an actionable implementation plan for the Orchestrator. | `.github/agents/planner.agent.md` |
| **Coder** | GPT-5.6 Luna (copilot) | Implements the dashboard logic and assigned application changes with clear, deterministic, testable code; it also validates runnable-app support when assigned. | `.github/agents/coder.agent.md` |
| **Designer** | GPT-5.6 Luna (copilot) | Defines and implements UI/UX direction within its assigned scope, including accessibility, information hierarchy, responsive behavior, visual clarity, and Project Pulse dashboard styling. | `.github/agents/designer.agent.md` |

The Orchestrator will typically obtain a plan from the Planner, delegate implementation and design work to the Coder and Designer, and then verify that the integrated dashboard meets the requirements. No custom agent stages, commits, or pushes changes; Git operations remain under the learner's control through Copilot CLI.
