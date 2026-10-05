# Agentic coding: role-based workflow for GitHub Copilot

Role instructions that give each GitHub Copilot chat in VS Code one job. For larger work, a strong model writes a plan, a cheaper model builds it, and a strong model reviews it. A read-only Chat role handles questions and research, and a Solo role handles small changes without a plan.

- `COPILOT-SOP.md`: setup and day-to-day procedure. Start here.
- `.github/copilot-instructions.md`: the role rules, loaded into every chat.
- `.github/agents/`: optional custom agents that pin a model per role. Replace the `model:` placeholders before use.
- `docs/plans/`: where plan files are saved.
- `docs/notes/`: where a Chat saves research notes.

To use it in your own project, copy `.github/`, `docs/plans/` and `docs/notes/` into that repository.
