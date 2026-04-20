# Exercise 2 — Parallel subagents with git worktrees

## Goal

Implement **four features in parallel** by asking Claude to spawn four subagents, each working in its own isolated git worktree, then integrate the four branches into one. You'll learn to orchestrate concurrent agent work with a single prompt, and to review and merge diffs that come back from independent agents.

## Why this technique

For independent feature work, running agents one at a time wastes wall-clock time. You can ask Claude to spawn **subagents** that each work in an **isolated git worktree** — a full checkout on a new branch, separate from your main working tree. Four subagents running at once gives you four parallel implementations. The catch: they'll all produce diffs on the same file (`index.html`). The shared spec in `exercises/02-specs.md` carves the file into non-overlapping ownership zones so the diffs can actually merge.

---

## What you're implementing

Four improvements from the repo's `README.md`, one per subagent:

1. **Persistence** — save / load todos via `localStorage`.
2. **Edit-in-place** — double-click a todo's text to edit it.
3. **Filter bar** — `All` / `Active` / `Completed` tabs.
4. **Clear completed** — button to remove all completed todos.

Full specifications — schema, function ownership, integration rules, per-feature requirements — are in **[`exercises/02-specs.md`](./02-specs.md)**. You don't need to modify that file; the subagents will read it.

> **Important:** these four features overlap with the README's improvements list by design. Do **not** pre-implement any of them on `main` — you want the agents doing the work.

### Why this resists one-shotting

- All four features touch `render()` and the `todos` array. Without the shared spec, four parallel agents produce four diffs that conflict on the same lines.
- Even with the spec, integration is a real task: reading four diffs, picking merge order, resolving semantic conflicts.
- Cleanup is easy to get wrong — deleting worktree directories with `rm` leaves dangling metadata.

---

## Setup

- Branch `main`, clean working tree.
- `git --version` ≥ 2.5 (any modern macOS is fine).
- Do **not** pre-implement any of the four features.

---

## Steps

### 1. Kick off with one prompt

Paste the following into Claude Code:

```text
Please spawn four subagents in parallel, each working in an isolated git
worktree on its own branch, to implement one feature each from
exercises/02-specs.md. All four agents must read @exercises/02-specs.md
first and follow the Shared Contract section exactly — especially the
schema, function ownership, and integration-point rules.

Assign one feature per agent:
  - Feature 1 — Persistence       → branch feat/persistence
  - Feature 2 — Edit-in-place     → branch feat/edit-in-place
  - Feature 3 — Filter bar        → branch feat/filter-bar
  - Feature 4 — Clear completed   → branch feat/clear-completed

Each agent edits only index.html, leaves a single commit with the message
prescribed in the spec file, verifies by opening index.html in a browser,
and reports back: (1) worktree path, (2) branch, (3) a 3-line summary of
what changed, (4) any spec ambiguities encountered.

Launch all four agents in parallel in a single step so they run concurrently.
```

Claude should respond with a single message that launches all four agents at once. If it launches them sequentially instead, stop it and re-prompt with emphasis on "in parallel, in a single step."

### 2. Ask Claude to review

Once the subagents finish, have Claude review their work against the spec:

```text
Review the four subagents' work against @exercises/02-specs.md.
For each feature, tell me whether the implementation matches the spec
and whether the Shared Contract was respected — schema, function
ownership, render() decomposition.

If everything looks good, say so and wait for me to ask for the merge.
If something needs re-work, name the branch and what's wrong — don't
fix it yet.
```

If Claude flags problems, decide whether to re-spawn that agent (point it at the specific spec section it broke) or have Claude patch the branch directly. If Claude is happy with all four, continue.

### 3. Merge and verify

```text
Merge these four feature branches into a new feature branch.

Resolve any conflicts as you go.

When done, open index.html in my default browser.
```

Walk through the app yourself to confirm all four features work together:

- Add a todo, toggle it, edit it in place, refresh the page (persistence round-trip).
- Switch between `All` / `Active` / `Completed`; the `N items left` counter reflects active-across-all regardless of filter.
- Click `Clear completed`; the button hides when nothing is completed.

### 4. Cleanup

```text
Remove all four worktrees we created for this exercise using
`git worktree remove` (not rm). Then run `git worktree list` and
confirm only the main worktree remains.
```

> Claude must not `rm -rf` a worktree directory — that leaves dangling metadata under `.git/worktrees/`. If it does, follow up with `git worktree prune`.

---

## Acceptance criteria

- [ ] Four worktrees were created, four feature branches each with one commit implementing its feature.
- [ ] All four features merged into a new feature branch and working simultaneously when `index.html` is opened in a browser.
- [ ] Worktrees cleaned up via `git worktree remove` (confirmed with `git worktree list`).

---

## Hints

- Capture each worktree path and branch from Claude's report — you'll need them for the review and cleanup steps.
- If Claude tries to run the agents sequentially (one tool call at a time), interrupt and re-prompt — "launch all four in parallel in a single step" is the whole point.
- If a diff clearly violates the spec (e.g., an agent touched `render()`'s body instead of its sub-render), don't try to fix it during integration — discard that branch and re-spawn just that agent with a pointer to the specific spec section it broke.
- The filter bar agent is the most likely to overreach, because filtering looks like a cross-cutting concern. The spec's `renderFilterBar()` + `renderTodoList()` split is what keeps it bounded.
