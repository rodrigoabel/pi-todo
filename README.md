# pi-todo

Persistent per-project todo tracker with a live TUI widget for [Pi](https://github.com/nicobailon/pi-coding-agent).

## Attribution

**The extension is not mine.** It was written by [mrg2400xx](https://github.com/mrg2400xx) and
published as `pi-todo@1.2.0` under the MIT license ([`LICENSE`](LICENSE), Copyright (c) 2026
mrg2400xx). All credit for the design and the implementation goes to them. I am not affiliated with
them, and I claim no authorship or ownership over this extension.

This repository is a **personal-use fork**, not a new project, not a rewrite and not a maintained
successor. The upstream repository (`github.com/mrg2400xx/pi-todo`) is no longer reachable (404), so
this copy exists only to keep **one small tweak** I wanted for my own setup somewhere I can install
it from. Everything outside that tweak is upstream 1.2.0, verbatim.

If you don't need the tweak, use the original project or the npm package `pi-todo` instead of this
fork.

## What It Does

- Registers a `todo` tool the LLM can call to manage a task list
- Stores tasks in `.pi/todo.json` in the project root (per-project, persistent)
- Shows a **live widget** above the editor with the current todo state
- Auto-loads existing todos on session start

## The one change in this fork

Only `src/render.ts` behaves differently, and only in the live widget:

**The widget only takes up screen space for work that is still open.**

1. **Completed tasks are no longer rendered as rows.** A task vanishes from the widget as soon as
   its status becomes `done`. Nothing is deleted: `.pi/todo.json`, `todo list` and tool results still
   include completed items, and `clear` still removes them.
2. **The summary line still counts them**, so finished work stays accounted for without costing a
   row: `1 pending · 1 active · 1 done`.
3. **When every task is done the box collapses into a single line:**

   ```txt
   TODO ✓ 3 done
   ```

   It does not disappear on its own — run `/todo clear` to drop completed items and remove the line.

Two packaging-only changes, no behavior involved:

- `typebox` moved from `dependencies` to `peerDependencies` (with `peerDependenciesMeta.optional`),
  so Pi's host copy is used instead of a second physical copy of the library.
- `package.json` metadata points at this repository.

Everything else is upstream 1.2.0, unchanged. The tweak is
[one commit on top of the verbatim upstream import](https://github.com/rodrigoabel/pi-todo/compare/b4ded78...main),
touching a single source file, `src/render.ts` (plus docs). It is a personal tweak, not an upstream
release.

## Installation

Install straight from this repository:

```bash
pi install git:github.com/rodrigoabel/pi-todo
```

Or clone it and install the local path:

```bash
git clone https://github.com/rodrigoabel/pi-todo ~/Proyectos/pi-todo
pi install ~/Proyectos/pi-todo
```

Or point `~/.pi/agent/settings.json` at the extension file:

```json
{
  "extensions": ["~/Proyectos/pi-todo/src/index.ts"]
}
```

Then restart Pi, or run `/reload`. No build step is needed: Pi loads the TypeScript
sources directly through its bundled `jiti`.

## Tool: `todo`

### Actions

| Action | Required Fields | Description |
|--------|----------------|-------------|
| `add` | `text` | Create a new todo item |
| `update` | `id` | Modify an item's text/status/priority/assignee/blockedBy |
| `toggle` | `id` | Cycle status: pending → in-progress → done → pending |
| `remove` | `id` | Delete an item |
| `list` | — | List all items (optional: `filter`) |
| `clear` | — | Remove all completed items |
| `reorder` | `id`, `direction` | Move an item up or down in the list |

### Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | `string` | yes | One of: add, update, toggle, remove, list, clear, reorder |
| `id` | `string` | conditional | Task ID (e.g. T-001) |
| `text` | `string` | conditional | Task description |
| `status` | `string` | no | pending, in-progress, done, blocked |
| `priority` | `string` | no | low, medium, high, critical (default: medium) |
| `assignee` | `string` | no | Who's doing it |
| `filter` | `string` | no | Filter for list: all, pending, in-progress, done, blocked |
| `blockedBy` | `string` | no | Task ID this is blocked by |
| `direction` | `string` | no | Reorder direction: up or down |

### Examples

```
todo({ action: "add", text: "Build StorefrontController", priority: "high", assignee: "Ultron" })
todo({ action: "toggle", id: "T-001" })
todo({ action: "list", filter: "pending" })
todo({ action: "reorder", id: "T-003", direction: "up" })
todo({ action: "clear" })
```

## Slash Commands

- `/todo` — List all todo items
- `/todo clear` — Clear completed items

## Storage

Tasks are stored in `.pi/todo.json` in the project root:

```json
{
  "items": [
    {
      "id": "T-001",
      "text": "Build StorefrontController",
      "status": "in-progress",
      "priority": "high",
      "assignee": "Ultron",
      "createdAt": "2026-07-05T19:02:00Z",
      "updatedAt": "2026-07-05T19:02:00Z",
      "blockedBy": null
    }
  ],
  "counter": 1
}
```

## Widget

The live widget appears above the editor inside a green Unicode box. Only **active**
tasks (`pending`, `in-progress`, `blocked`) get a row:

```txt
┌───────────────────────────────────────────────────┐
│ TODO · 2 tasks                                     │
│   1 pending · 1 active · 1 done                    │
│   [ ] [HIGH] Build StorefrontController · Ultron   │
│   [•] [LOW] Verify storefront routes · Quinn       │
└───────────────────────────────────────────────────┘
```

The `1 done` in the summary line is a task that is already finished: counted, but
not listed. At most 8 rows are shown; the rest collapse into `… and N more`.

When there is nothing left to display, the box goes away and a single line
remains:

```txt
TODO ✓ 3 done
```

Status icons: `[ ]` pending, `[•]` in-progress, `[x]` done, `[!]` blocked. In this
fork `[x]` only appears in tool results and `/todo`, never as a widget row.

## Changelog

### v1.2.1 — personal tweak, not an upstream release

- **Completed tasks no longer take up widget rows.** `done` items are filtered out of the live
  widget (`visibleItems()` in `src/render.ts`). The store and tool results keep them, so nothing is
  lost and `clear` behaves as before.
- **All-done collapse.** When every task is done, the bordered box is replaced by a single line:
  `TODO ✓ 3 done`.
- `typebox` declared as an optional peer dependency instead of a bundled dependency.

### v1.2.0

- **ASCII status icons** — Replaced emoji status icons with ASCII equivalents (`[ ]`, `[•]`, `[x]`, `[!]`) for universal terminal compatibility and cleaner rendering.
- Removed clipboard emoji from widget header.

### v1.1.0

- **Bordered widget** — The live widget now renders inside a green Unicode box border (uses `theme.fg("success")`) for better visual separation from the chat area.

### v1.0.0

- Initial release: todo tool, live widget, slash commands, per-project persistence.

## Design Decisions

- **Per-project, not global** — Each project has its own todo list
- **Main agent only** — Subagents cannot call the todo tool
- **Auto-generated IDs** — T-001, T-002, T-003...
- **Synchronous file I/O** — Avoids race conditions (Node single-threaded)
- **No external dependencies** — Uses only Pi's built-in typebox
