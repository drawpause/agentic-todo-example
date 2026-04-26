Modify the current plan to use Claude Code's Agent Teams feature (TeamCreate) for parallel execution.

## Step 1: Read the plan

Read the current plan file from the plan file path provided in the system context.

If **no plan file exists**, tell the user:
> "No active plan found. Create a plan first (e.g. with /plan), then run /agentify to parallelize it with Agent Teams."

Then **stop**.

## Step 2: Assess parallelizability

Evaluate whether the plan benefits from Agent Teams. A plan is **NOT a good candidate** if:

- It has fewer than 3 distinct, independent work streams
- All tasks are tightly sequential (each step depends on the previous one)
- The total scope is small enough that a single session can finish quickly
- The work touches the same files across tasks (high merge-conflict risk)

If the plan is not a good candidate, tell the user **why** and **stop**. Example:
> "This plan has only 2 steps that are sequential — Agent Teams won't help here. Teams shine when you have 3+ independent work streams that can run in parallel."

## Step 3: Remind about tmux

Before modifying the plan, remind the user:

> Make sure you are running this session inside tmux. Agent Teams uses tmux split panes to display teammates — without it, teammate output won't be visible.

Wait for the user to acknowledge before proceeding.

## Step 4: Identify work streams and team structure

Analyze the plan and identify:

1. **Independent work streams**: groups of tasks that can run in parallel with minimal file overlap
2. **Sequential phases**: groups of work streams where one phase must complete before the next starts (run separate team sessions per phase)
3. **Dependencies between streams**: shared files, config, or interfaces that need coordination
4. **Teammate count per phase**: aim for 3-5 teammates per team session (sweet spot for coordination overhead vs parallelism)

For each teammate, identify:
- A short descriptive name (used as the teammate name and worktree name)
- The specific files they will create or modify
- Reference files they should read before starting
- Whether they overlap with any other teammate's files (flag these — they need special handling)

## Step 5: Rewrite the plan for Agent Teams

Modify the plan file to include the following structure. **Emphasize the correct tool sequence** — this is the most common failure mode:

### For each phase/team session, add:

**Launch section** with:
1. The **exact tool sequence** that MUST be followed:
   - Step 1: `TeamCreate` with `team_name="<descriptive-name>"`
   - Step 2: Read the plan file and any reference files
   - Step 3: `TaskCreate` (one per teammate's work)
   - Step 4: `Agent` with **`team_name`** parameter (MUST include this — without it, regular subagents are created instead of teammates), plus `model="sonnet"`, `mode="plan"`, and `name=<teammate-name>`
   - Step 5: `TaskUpdate` to assign tasks to teammates

2. A **bold warning** about the most critical requirement:
   > **CRITICAL: Every `Agent` call MUST include `team_name`. Without it, Claude creates regular subagents (no shared task list, no messaging, no tmux panes) instead of real teammates. This is the #1 failure mode.**

3. **Git isolation instructions**: each teammate MUST work in its own git worktree:
   - Worktree named after the teammate
   - Each gets its own branch (`worktree-<name>`) and working directory
   - Zero risk of git conflicts between teammates

4. **Per-teammate specifications**:
   - Name and worktree name
   - Files to create/modify (must not overlap with other teammates)
   - Reference files to read first
   - Specific instructions and acceptance criteria
   - Commit rules: clean history, logical commits, ticket ID in messages, imperative mood, no WIP commits

5. **Quality gates for plan approval** (since teammates use `mode="plan"`):
   - What the lead should look for before approving a teammate's plan
   - Patterns or conventions the teammate must demonstrate understanding of

**Integration section** with:
1. How the lead merges teammate branches (order matters if there are dependencies)
2. Integration tasks the lead handles after merging (wiring things together, updating __init__.py files, etc.)
3. Verification steps (run tests, linting, etc.)
4. Cleanup: tell lead to clean up the team, verify no orphaned tmux sessions

### Dependency handling

If work streams have dependencies (e.g., one teammate produces config that another needs):
- **Option A (simpler)**: Let all start in parallel; the lead reconciles at merge time. Works when the plan is detailed enough that both converge on compatible interfaces.
- **Option B (safer)**: Use task dependencies so the dependent teammate starts only after the dependency is ready, then copies/cherry-picks the needed files into its worktree.

Document which option you chose and why.

## Step 6: Add steering guide

Append a "Steering During Execution" section to the plan with:

- How to monitor progress (tmux panes or Shift+Down)
- Common interventions table:
  | Situation | Action |
  |---|---|
  | Lead used Agent without team_name | Stop, clean up, re-prompt. Check ~/.claude/teams/ — if missing, TeamCreate was never called. |
  | Teammate didn't create worktree | Message them: "Create a worktree named X and work inside it" |
  | Teammate diverges from conventions | Message them with specific file:line references to follow |
  | Lead starts implementing instead of waiting | Tell lead: "Wait for teammates to complete before proceeding" |
  | Teammate stuck on error | Click into their pane and give guidance |
- Cleanup checklist: team cleanup, orphaned tmux sessions, worktree removal, branch cleanup

## Step 7: Review

Present a summary to the user:
- Number of phases / team sessions
- Teammates per phase
- Key dependencies or risks
- Estimated cost multiplier (N teammates = roughly Nx single session)

Ask the user to review the modified plan before executing it.
