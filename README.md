# Agentic coding: role-based workflow for GitHub Copilot

Role instructions that give each GitHub Copilot chat in VS Code one job. For larger work, a strong model writes a plan, a cheaper model builds it, and a strong model reviews it. A read-only Chat role handles questions and research, and a Solo role handles small changes without a plan.

- `COPILOT-SOP.md`: setup and day-to-day procedure. Start here.
- `SETUP.md`: steps a Copilot agent follows to install the workflow into a project.
- `.github/copilot-instructions.md`: the role rules, loaded into every chat.
- `.github/agents/`: optional custom agents, one per role. Pinning a model to each is optional.
- `docs/plans/`: where plan files are saved.
- `docs/notes/`: where a Chat saves research notes.

To use it in your own project, open a Copilot chat in Agent mode there and paste:

```text
Follow the setup steps in https://github.com/michael-z-liu/Agentic-Coding-Guide/blob/main/SETUP.md
```
