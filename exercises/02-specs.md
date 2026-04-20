# Feature Specs — Parallel Worktree Exercise

This file is the single source of truth for four parallel feature implementations of the TODO app in `index.html`. Each subagent implements **exactly one feature** and must follow the **Shared Contract** section below. No agent may modify a function owned by another feature.

---

## Shared contract

### Schema

Each todo is an object with this shape:

```js
// { id: string, text: string, completed: boolean, createdAt: number }
```

- Generate `id` with `crypto.randomUUID()` on creation.
- Set `createdAt` to `Date.now()` on creation.
- Never mutate `id` or `createdAt` after creation.

### Function ownership

- **Persistence** owns `loadTodos()` and `saveTodos()`. It calls `saveTodos()` from every mutation point (add / toggle / delete / edit / clear).
- **Edit-in-place** owns `enterEditMode(id)` and `exitEditMode(save)`, plus any edit-specific CSS.
- **Filter bar** owns `renderFilterBar()` and the `currentFilter` state variable (one of `"all"`, `"active"`, `"completed"`).
- **Clear completed** owns `clearCompleted()` and `renderClearCompletedButton()`.

No feature may modify a function owned by another feature.

### Integration point

Decompose the existing monolithic `render()` into named sub-renders. The top-level `render()` stays as this exact call sequence — no logic inside it:

```js
function render() {
  renderFilterBar();
  renderTodoList();
  renderClearCompletedButton();
  renderRemaining();
}
```

Each feature owns at most one sub-render plus its own helpers. `renderTodoList()` and `renderRemaining()` are the existing rendering logic split out — whichever feature lands first creates them.

### State conventions

- New top-level state variables go right after `var todos = [];` at the top of the `<script>` block.
- camelCase for variable and function names. No abbreviations.
- Persistence hooks fire on every mutation, without exception.

### Commit convention

Each agent leaves a single commit on its worktree branch with message:

```text
feat(<feature-slug>): <short summary>
```

---

## Feature 1 — Persistence

- **Branch:** `feat/persistence`
- **Slug for commit:** `persistence`

Requirements:
- Load todos from `localStorage` key `"todos.v1"` on page load.
- If the key is missing or `JSON.parse` throws, start with an empty array.
- Save on every mutation (add, toggle, delete, edit, clear completed).
- Persistence functions must not touch the DOM or call `render()`.

---

## Feature 2 — Edit-in-place

- **Branch:** `feat/edit-in-place`
- **Slug for commit:** `edit-in-place`

Requirements:
- Double-clicking a todo's text replaces the text span with an `<input>` pre-filled with the current text and focused.
- Pressing `Enter` commits the edit. Pressing `Escape` reverts. Clicking away also commits.
- If the committed text is empty (after trim), delete the todo.
- Only one todo can be in edit mode at a time.

---

## Feature 3 — Filter bar

- **Branch:** `feat/filter-bar`
- **Slug for commit:** `filter-bar`

Requirements:
- Three buttons above the todo list: `All`, `Active`, `Completed`.
- The active button is visually distinct (outlined, highlighted, or similar).
- Clicking a button sets `currentFilter` and re-renders.
- The list shows only todos matching the current filter.
- The `N items left` counter always reflects active-across-all-todos, regardless of the current filter.

---

## Feature 4 — Clear completed

- **Branch:** `feat/clear-completed`
- **Slug for commit:** `clear-completed`

Requirements:
- Button labelled `Clear completed`, rendered below the list via `renderClearCompletedButton()`.
- Clicking it removes all completed todos from the array.
- The button is hidden when no todos are completed.
