# SOP: Plan / Build / Review workflow in GitHub Copilot

## What you are setting up

- One always-loaded file tells every chat how to work out its role (Planner, Builder, Reviewer) and what that role may and may not do.
- A plan file in `docs/plans/` is the handoff between chats. Each chat writes what the next one needs into it.
- You use an expensive model for the planning and review chats and a cheap model for the build chat.
- Each chat ends by printing the exact command to paste into the next chat.

Copilot does not switch models by itself from the instruction file alone. Either you pick the model in the model picker (Method A), or you let custom agents pin it for you (Method B).

## One-time setup (about 5 minutes)

1. In the repository root, create the folder `.github` if it does not exist.
2. Copy `copilot-instructions.md` to `.github/copilot-instructions.md`.
3. Create the folder `docs/plans/`.
4. Optional but recommended: create `.github/agents/` and copy `planner.agent.md` and `builder.agent.md` into it.
5. Open each agent file and replace the `model:` placeholder with the model name exactly as it appears in your Copilot model picker.
   - Planner: your strongest reasoning model.
   - Builder: a cheaper or included (low multiplier) model.
6. Commit the files so your team gets the same behaviour.
7. Test: open a new chat and paste the line below. The reply must start with `Role: PLANNER`. If it does not, see Troubleshooting.

   ```text
   PLAN: say which role you are
   ```

## How handoff works

When a chat finishes, or has to stop, its last lines are a handoff: a line starting with `Next:` that tells you which model to select, then the command in its own code block. It looks like this:

````markdown
Next: open a new chat, select your strong model, and paste this.

```text
REVIEW: docs/plans/PLAN-add-login-rate-limit.md
```
````

To hand off:

1. Click the copy button on the code block.
2. Open a new chat. Do not reuse the current one.
3. Select the model the `Next:` line names.
4. Paste and send.

The command is always alone in its own code block, never inside a sentence, so the copy button gives you exactly the text to paste. If a chat ever prints a command inside a paragraph, see Troubleshooting.

What each chat prints when it finishes:

| Chat | Outcome | Command it prints | Model for the next chat |
|---|---|---|---|
| Planner | Plan drafted | None. It asks you to approve or request changes. | |
| Planner | You approve | `BUILD:` + plan path | Cheap |
| Builder | All steps done | `REVIEW:` + plan path | Strong |
| Builder | Blocked on a step | `PLAN:` + plan path | Strong |
| Reviewer | PASS or PASS WITH NOTES | None. You check the diff, run the tests and commit. | |
| Reviewer | FAIL, code defects | `BUILD:` + plan path | Cheap |
| Reviewer | FAIL, plan or design is wrong | `PLAN:` + plan path | Strong |

The plan file carries the detail between chats. The Builder records progress and blocks in its Build log, and the Reviewer records defects in its Review log, so the next chat picks them up from the file.

## Method A: keywords

### Phase 1: Plan
1. Open a new chat. Use Agent mode so Copilot can read the code and save the plan file.
2. In the model picker, select your strong model.
3. Type `PLAN:` followed by what you want, any constraints, and what "done" means. For example:

   ```text
   PLAN: add rate limiting to the login endpoint, 5 attempts per minute per IP, with tests
   ```

4. Answer any clarifying questions.
5. Copilot saves `docs/plans/PLAN-<slug>.md` with `Status: DRAFT`. If it only prints the plan, save it to that path yourself.

### Phase 2: Approve
6. Read the plan. Ask for changes in the same chat until you are happy.
7. Reply with the line below. The Planner sets `Status: APPROVED` and prints the `BUILD:` command.

   ```text
   approve
   ```

### Phase 3: Build
8. Follow the handoff: new chat, cheap model, Agent mode, paste the `BUILD:` command.
9. Let it work through the steps. It ticks each step in the plan file as it goes.
10. When it finishes it prints the `REVIEW:` command.
11. If it stops with "blocked", it prints a `PLAN:` command instead. Paste that into a new chat with your strong model. The Planner fixes the plan, you approve it, and the next Builder resumes from the first unticked step.

### Phase 4: Review
12. Follow the handoff: new chat, strong model, paste the `REVIEW:` command.
13. On PASS, go to Phase 5.
14. On FAIL, it prints a `BUILD:` command (code defects) or a `PLAN:` command (plan problem). Follow it. After the fix, the Builder prints `REVIEW:` again.

### Phase 5: Close
15. Review the diff yourself, run the tests, commit.

## Method B: custom agents (model is pinned automatically)

1. Open a new chat and select **Planner** in the agent dropdown. The strong model is selected for you.
2. Describe the task. No `PLAN:` keyword is needed.
3. Review the plan and reply `approve`.
4. Open a new chat, select **Builder**, and paste the `BUILD:` command the Planner printed. (The **Build this plan** button also works, but it carries the planning conversation into the build, which costs more.)
5. For review, open a new chat in the normal Agent mode with your strong model and paste the `REVIEW:` command.

The printed commands work the same way with custom agents. If the selected agent and the keyword disagree (for example **Planner** selected and a `BUILD:` command pasted), the chat does no work and asks you to switch.

## What happens when you ask the wrong chat

Each chat keeps one role for its whole life. If you ask it for another role's work, it does none of it, tells you which role the work belongs to, and prints the command for the right chat.

| You are in | You ask for | What it does |
|---|---|---|
| Planner | A review of the plan itself | Does it. Revising the plan is Planner work. |
| Planner | A review of the built code | Refuses, prints the `REVIEW:` command. |
| Planner | Code changes | Refuses, prints the `BUILD:` command. |
| Builder | A change to the plan's scope, steps or design | Refuses, prints the `PLAN:` command. |
| Builder | A review of its own build | Refuses, prints the `REVIEW:` command. |
| Reviewer | A fix for a defect it found | Refuses, records the defect in the plan file, prints the `BUILD:` command. |
| Any | An explanation of the code or plan | Answers briefly. Nothing is changed. |

It also checks that the plan is ready for the step you asked for:

| Situation | What it does |
|---|---|
| You ask for a code review but the plan is not built yet | Says so and prints the `BUILD:` command instead. |
| `BUILD:` or `REVIEW:` with no plan path, or a path that does not exist | Stops, lists the files in `docs/plans/` and asks which one. |
| `BUILD:` on a plan that is still `DRAFT` | Asks whether you have approved it. |
| `BUILD:` on a plan that is `BLOCKED` | Stops and prints the `PLAN:` command. |
| `BUILD:` on a plan that is already built | Does not rebuild. Prints the `REVIEW:` command. |
| `REVIEW:` on a plan that is already reviewed | Asks whether you want a second review. |
| `PLAN:` naming an existing plan file | Revises that plan instead of writing a new one. |
| First message has no role keyword | Asks which role the chat is. Answer with one word. |

## Plan status reference

| Status | Meaning | Next chat |
|---|---|---|
| `DRAFT` | Written or revised, waiting for your approval | Stay in the Planner chat |
| `APPROVED` | Ready to build | Builder |
| `BLOCKED` | Plan needs fixing | Planner |
| `BUILT` | All steps done and verified | Reviewer |
| `CHANGES REQUESTED` | Review found code defects | Builder |
| `REVIEWED` | Review passed | None. Commit. |

## Rules of thumb

- One task, one plan file. One role per chat.
- Never build in the planning chat. The saving comes from the cheap model doing the build with a fresh, small context.
- Skip planning for trivial edits (rename, typo, one-line fix). Use a normal chat with the cheap model.
- If the Builder gets blocked twice on the same plan, the plan is too vague. Re-plan with more detail rather than switching the Builder to a stronger model.
- Keep finished plans in `docs/plans/` as a record, or delete them after merge if your team prefers.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Reply does not start with `Role: ...` | Check the file is exactly `.github/copilot-instructions.md` at the repo root, and that instruction files are enabled in your Copilot settings. Start a new chat. |
| No handoff at the end of a reply | Ask "print the handoff". If it keeps happening, the chat is too long; start a new chat with the same command. |
| Command printed inside a sentence, or with a placeholder instead of the real path | Ask "print the handoff command alone in a code block with the real path". |
| A chat does another role's work instead of refusing | Stop it, undo the changes, and start a new chat with the right keyword. Cheaper models drift more in long chats. |
| Planner starts editing code | Say "You are PLANNER, revert and stop." The Planner agent's instructions restrict writes to `docs/plans/`. |
| Planner cannot save the plan or set the status | You are in Ask mode. Switch to Agent mode, or make the change by hand. |
| Agents do not appear in the dropdown | Files must be in `.github/agents/` and end in `.agent.md`. Your organisation may also have custom agents disabled. |
| Agent ignores the `model:` line | The name must match the model picker exactly. If it still fails, pick the model by hand. |
| Builder drifts from the plan | Stop it and start a new chat with the same `BUILD:` command. It resumes from the first unticked step. |
