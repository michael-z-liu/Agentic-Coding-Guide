# Setup instructions for the agent

These instructions are for a GitHub Copilot agent installing this workflow into a project. A person can follow them by hand as well.

Source repository: https://github.com/michael-z-liu/Agentic-Coding-Guide

## Steps

1. Confirm the workspace is the root of the project the workflow is going into. If you cannot tell, ask.

2. Get these files from the source repository. Raw files are at `https://raw.githubusercontent.com/michael-z-liu/Agentic-Coding-Guide/main/<path>`. If you cannot fetch them, stop and ask the user to download the repository and tell you where it is.

   | Source path | Save in the project as |
   |---|---|
   | `.github/copilot-instructions.md` | `.github/copilot-instructions.md` |
   | `.github/agents/planner.agent.md` | `.github/agents/planner.agent.md` |
   | `.github/agents/builder.agent.md` | `.github/agents/builder.agent.md` |
   | `.github/agents/chat.agent.md` | `.github/agents/chat.agent.md` |
   | `.github/agents/solo.agent.md` | `.github/agents/solo.agent.md` |
   | `COPILOT-SOP.md` | `docs/COPILOT-SOP.md` |

3. Never overwrite silently. If a destination file already exists, show the user which one and ask: replace it, keep it, or (for `copilot-instructions.md` only) append the workflow to the end of the existing file.

4. Copy the files exactly. Do not reword, shorten or reformat them.

5. Create the folders `docs/plans/` and `docs/notes/`, each with an empty `.gitkeep` file.

6. Model pinning. The agent files come with no `model:` line, which means each chat uses whichever model is selected in the model picker.

Ask this once, and wait for the answer:
   
   "Do you want to pin a model to each role now? Reply `pin` or `skip`. If you skip, each chat uses whichever model is selected in the model picker, and you can pin models later."
   
   - On `skip`: make sure no agent file has a `model:` line, and move on. Do not ask again.
   - On `pin`: ask for two model names, typed exactly as they appear in the Copilot model picker: a strong model (used by Planner) and a cheap model (used by Builder, Solo and Chat). The user may give only one; pin only the roles it covers. Then add a line of the form `model: ['<name>']` directly under the `description:` line of each matching file in `.github/agents/`, replacing any existing `model:` line.
   - Never guess, suggest or invent a model name. Use only names the user typed.

7. Do not commit. Finish with a list of the files you created or changed, and whether models were pinned. Then tell the user to open a new chat and paste the line below; the reply should start with `Role: PLANNER`.

   ```text
   PLAN: say which role you are
   ```

## Changing models later

Once the workflow is installed, start a new chat with one of these:

```text
SETUP: pin models
```

```text
SETUP: remove the pinned models
```
