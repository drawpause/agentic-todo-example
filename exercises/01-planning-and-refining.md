# Exercise 1 — Planning with `/plan` and `/refine-plan`

## Goal

Implement a non-trivial feature on the TODO app by **first producing a plan, then hardening it with the `/refine-plan` skill, and only then writing code**. You'll learn to recognise what an LLM's first-draft plan tends to miss, and to iterate on a plan the way you'd iterate on code.

## Why this technique

Complex features fail when the agent jumps straight to coding. A good plan surfaces schema changes, focus/mode interactions, and ripple effects before a single line of JavaScript is written. `/refine-plan` is a custom skill that self-critiques the plan, scores it 0–10, rewrites the weakest sections, and re-scores — looping on its own until the plan is solid. You'll learn more from one `/refine-plan` run than from five rounds of fix-the-code.

---

## Setup — install the `/refine-plan` skill

This repo bundles the skill so the exercise is self-contained.

```bash
mkdir -p ~/.claude/skills/refine-plan
cp exercises/skills/refine-plan/SKILL.md ~/.claude/skills/refine-plan/SKILL.md
```

**Restart Claude Code** if it's already running — skills are loaded at startup, so a session that was running before the install won't see the new skill.

Verify: in Claude Code, type `/` and confirm `refine-plan` appears in the skill list. (If it was already installed, the `cp` is a harmless overwrite.)

---

## Steps

### 1. Draft the initial plan

Enter plan mode (Shift+Tab twice, or run `/plan`) and paste the **Target feature** description below as your prompt. Let Claude explore the codebase and write the plan to the plan file it reports. **Do not approve it yet.**

### 2. Refine

When plan mode finishes and shows you the approval prompt, **hit `Esc` first** — that returns you to the normal input so you can type a slash command. Then run `/refine-plan` **once**. The skill iterates on its own: it scores the plan 0–10, researches weak areas, rewrites weak sections, re-scores, and repeats until the score reaches 8+ with no blockers, or until further progress is blocked. You don't re-invoke it — watch it loop and read the final report.

Read the skill's final report carefully. Notice the starting vs. final score, and which weakness categories came up most — that's the lesson.

### 3. Execute

Approve the plan and have Claude implement it. Observe the outcome.

---

## Target feature

Use the block below as your plan-mode prompt, verbatim:

```text
Convert the single-list TODO app in index.html into a multi-list project manager
with all of the following features working together:

1. Named lists — a sidebar showing all lists with add / rename / delete.
2. Per-todo tags — comma-separated tags on each todo, displayed as chips.
3. Due dates — optional due date per todo; overdue todos are visually distinct
   (colour, icon, or both).
4. Search box — filters the active list's todos by free text AND by tag.
5. Keyboard shortcuts:
     n         focus the new-todo input
     /         focus the search box
     j / k     move selection down / up
     x         toggle complete on the selected todo
     d         delete the selected todo
     1 .. 9    switch to the Nth list
   Shortcuts must NOT fire while the user is typing in a text input.
6. localStorage persistence — full state survives a hard refresh, including
   which list was active.

Constraints:
- UI stays as a single index.html file. No build tooling, no frameworks.
- Opening index.html directly in a browser must work.
- Persisted state must carry a schemaVersion field for future migrations.
```

### Why this resists one-shotting

- The data model has to change from a flat `todos` array to something like `{ schemaVersion, activeListId, lists: [{ id, name, todos: [...] }] }` — **every existing function breaks**.
- Keyboard shortcuts must **not** fire while the user is typing in the new-todo input or search box. It's easy to forget this on a first pass.
- Search + tag filter + active-list selection compose into the render pipeline as three filter stages — the ordering matters.
- The existing code uses inline `onclick="addTodo()"` and global functions (see `index.html:103` and `index.html:117-125`). A good plan consciously decides whether to preserve or refactor this style rather than drifting into a half-refactor.

---

## Acceptance criteria

- [ ] All 6 features work end-to-end in a browser when `index.html` is opened directly (no server needed).
- [ ] `localStorage` state survives a hard refresh, including active-list selection.
- [ ] Typing in the new-todo input or search box does **not** trigger `j`, `k`, `x`, `d`, or `n` shortcuts.
- [ ] Overdue todos are visually distinct from non-overdue ones.

---

## Hints

- The most common miss in plans for this exercise is the **focus rule** for keyboard shortcuts. If your plan doesn't say *explicitly* what happens when a text input is focused, that's a weakness.
- A schema version field (`schemaVersion: 1`) is cheap now and prevents pain later when you'd otherwise need a silent migration.
- Don't refactor the inline `onclick` style unless your plan says you will — drift between "planned" and "shipped" is what this exercise is training you to notice.
