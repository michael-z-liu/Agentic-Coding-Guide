# SOP: Plan / Build workflow in GitHub Copilot

## What you are setting up

- One always-loaded file tells every chat how to work out its role (Planner, Builder, Reviewer) and what that role may and may not do.
- A plan file in `docs/plans/` is the handoff between chats.
- You use an expensive model for the planning chat and a cheap model for the build chat.

Copilot does not switch models by itself from the instruction file alone. Either you pick the model in the model picker (Method A), or you let custom agents pin it for you (Method B, VS Code).

## One-time setup (about 5 minutes)

1. In the repository root, create the folder `.github` if it does not exist.
2. Copy `copilot-instructions.md` to `.github/copilot-instructions.md`.
3. Create the folder `docs/plans/`.
4. VS Code only, optional but recommended: create `.github/agents/` and copy `planner.agent.md` and `builder.agent.md` into it.
5. Open each agent file and replace the `model:` placeholder with the model name exactly as it appears in your Copilot model picker.
   - Planner: your strongest reasoning model.
   - Builder: a cheaper or included (low multiplier) model.
6. Commit the files so your team gets the same behaviour.
7. Test: open a new chat and type `PLAN: say which role you are`. The reply must start with `Role: PLANNER`. If it does not, see Troubleshooting.

## Method A: keywords (works in any IDE with Copilot Chat)

### Phase 1: Plan
1. Open a new chat. Use Agent mode if available, so Copilot can read the code and save the plan file.
2. In the model picker, select your strong model.
3. Type: `PLAN: <what you want, constraints, definition of done>`
4. Answer any clarifying questions.
5. Copilot saves `docs/plans/PLAN-<slug>.md`. If it only prints the plan, save it to that path yourself.

### Phase 2: Approve
6. Read the plan. Ask for changes in the same chat until you are happy.
7. Change `Status: DRAFT` to `Status: APPROVED` in the plan file.

### Phase 3: Build
8. Open a new chat. Do not reuse the planning chat.
9. In the model picker, select your cheap model. Use Agent mode.
10. Type: `BUILD: docs/plans/PLAN-<slug>.md`
11. Let it work through the steps. It ticks each step in the plan file as it goes.
12. If it replies "Blocked at step n", go back to the plan chat (or a new `PLAN:` chat), fix the plan, then start a new `BUILD:` chat. It resumes from the first unticked step.

### Phase 4: Review (optional)
13. Open a new chat, select your strong model.
14. Type: `REVIEW: docs/plans/PLAN-<slug>.md`
15. On FAIL, send the defects to a new `BUILD:` chat (small fixes) or `PLAN:` chat (design problems).

### Phase 5: Close
16. Review the diff yourself, run the tests, commit.

## Method B: custom agents (VS Code, model is pinned automatically)

1. Open a new chat and select **Planner** in the agent dropdown. The strong model is selected for you.
2. Describe the task. No `PLAN:` keyword needed.
3. Review the plan and set `Status: APPROVED`.
4. Either click the **Build this plan** button, or (cheaper, cleaner context) open a new chat, select **Builder**, and type the plan file path.
5. Review and commit as in Method A.

## Rules of thumb

- One task, one plan file, one build chat.
- Never build in the planning chat. The whole saving comes from the cheap model doing the build with a fresh, small context.
- Skip planning for trivial edits (rename, typo, one-line fix). Just use a normal chat with the cheap model.
- If the builder gets blocked twice on the same plan, the plan is too vague. Re-plan with more detail rather than switching the builder to a stronger model.
- Keep finished plans in `docs/plans/` as a record, or delete them after merge if your team prefers.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Reply does not start with `Role: ...` | Check the file is exactly `.github/copilot-instructions.md` at the repo root, and that instruction files are enabled in your Copilot settings. Start a new chat. |
| Planner starts editing code | Say "You are PLANNER, revert and stop." In VS Code, use the Planner agent, whose instructions restrict writes to `docs/plans/`. |
| Planner cannot save the plan | You are in Ask mode. Switch to Agent mode, or save the printed plan yourself. |
| Agents do not appear in the dropdown | Files must be in `.github/agents/` and end in `.agent.md`. Your organisation may also have custom agents disabled. |
| Agent ignores the `model:` line | The name must match the model picker exactly. If it still fails, pick the model by hand. |
| Builder drifts from the plan | Stop it, start a new `BUILD:` chat. Long chats drift more. |
