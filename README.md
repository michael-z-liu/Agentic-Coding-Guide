# Agentic coding: Plan / Build / Review workflow for GitHub Copilot

Role instructions that make each GitHub Copilot chat in VS Code act as a Planner, Builder or Reviewer, so a strong model writes the plan and a cheaper model implements it.

- `COPILOT-SOP.md`: setup and day-to-day procedure. Start here.
- `.github/copilot-instructions.md`: the role rules, loaded into every chat.
- `.github/agents/`: optional custom agents that pin a model per role. Replace the `model:` placeholders before use.
- `docs/plans/`: where plan files are saved.

To use it in your own project, copy `.github/` and `docs/plans/` into that repository.
