@[assistant]:


@[assistant]:
I'll invoke the exploration skill first, then start the work.

@[assistant]:


@[assistant]:


@[assistant]:
I'll start by reading the issue and exploring the codebase in parallel.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
No git remote configured. Let me check for a local issue description and explore the codebase directly.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Good — there's already a `priority` reference in `format-task.ts`. Let me read all the source files.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Now let me check for tests and the package config.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
I now have a complete picture. Here's the structured exploration summary:

---

## Exploration Summary: Issue #50 — Add priority field to tasks with custom rules

### What Exists

| File | Role |
|---|---|
| `src/models/task.ts` | `Task` interface (id, title, status, createdAt, updatedAt) — **no priority field yet** |
| `src/models/task.ts` | `TaskFilter` interface (status, search) — **no priority filter yet** |
| `src/utils/format-task.ts` | `formatTask()` — shared display formatter, **does not show priority** |
| `src/utils/format-task.ts:16` | `derivePriority()` — **partial implementation exists**, only returns `'high'` for `in_progress`, `'medium'` otherwise. Explicitly notes it "does not support custom priority rules" |
| `src/services/task-service.ts` | `addTask(title)` — creates tasks with no priority param; `listTasks(filter)` — no priority filtering |
| `src/cli/index.ts` | CLI with `add` and `list` commands — no priority support |
| `src/api/index.ts` | HTTP handler (`GET /tasks`, `POST /tasks`) — no priority support |
| `src/reports/summary.ts` | Report generator using `formatTask` — no priority display |

### What Needs to Change

1. **`src/models/task.ts`** — Add `priority: 'high' | 'medium' | 'low'` field to `Task` interface; optionally add priority to `TaskFilter`
2. **`src/services/task-service.ts`** — Accept priority in `addTask()`, apply priority filtering in `listTasks()`
3. **`src/utils/format-task.ts`** — Extend `derivePriority()` to support custom rules; update `formatTask()` to display priority
4. **`src/cli/index.ts`** — Accept `--priority` flag on `add` command; possibly filter by priority on `list`
5. **`src/api/index.ts`** — Accept `priority` in POST body; possibly filter by priority on GET
6. **Tests** — No tests exist yet. New tests needed for all changed modules.

### All Consumers of Shared Interfaces

**`Task` interface** — imported by every source file (6 files total):
- `src/services/task-service.ts`
- `src/utils/format-task.ts`
- `src/cli/index.ts`
- `src/api/index.ts`
- `src/reports/summary.ts`

**`formatTask()`** — 3 call sites:
- `src/services/task-service.ts:31` (via `getFormattedTasks`)
- `src/cli/index.ts:10`
- `src/api/index.ts:15`
- `src/reports/summary.ts:9`

**`derivePriority()`** — 0 call sites (defined but **never called**)

### Key Decisions

1. **Stored vs. derived priority** — Should `priority` be an explicit field on `Task` (set at creation) or always derived from rules? The partial `derivePriority()` suggests a derived approach, but "custom rules" implies user-configurable logic. Likely need both: a stored default that can be overridden, or a rules engine.
2. **What are "custom rules"?** — Issue title says "custom rules" but we don't have the issue body. Need to clarify: are these user-defined rules (e.g., "tasks with 'urgent' in title get high priority"), or just richer derivation logic?
3. **`derivePriority` fate** — Currently unused. Either integrate it as the default rule, or replace it entirely with the new system.
4. **Breaking change to `Task`** — Adding a required `priority` field changes every place that constructs a `Task`. If optional, consumers must handle `undefined`.

### Risks

- **Blast radius** — `Task` is the core type; every source file imports it. Adding a required field breaks compilation everywhere until all construction sites are updated.
- **No existing tests** — Zero test files exist. Any refactoring has no safety net. Writing tests first (TDD) is strongly recommended.
- **`derivePriority` is dead code** — It was never integrated. Extending it without understanding why it was left unused could lead to wasted effort.
- **Both entry points** — Per AGENTS.md: "Changes to services or models must be tested through both entry points" (CLI and API).