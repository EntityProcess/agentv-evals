@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Running blast radius check for `Task` interface and `formatTask` function.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
The diff against master is empty — the branch has no uncommitted or committed changes yet to `task.ts` or `format-task.ts`. But based on the current state of the code and the branch name `feat/42-add-priority`, here's the blast radius analysis for consumers of `Task` and `formatTask`:

## Blast Radius: `Task` interface

**File** | **Usage** | **Status**
---|---|---
`src/services/task-service.ts` | Creates `Task` objects (line 7), stores them (line 4), returns them (lines 6, 18) | Consumer — must align with any new fields
`src/utils/format-task.ts` | Takes `Task` param in `formatTask` and `derivePriority` | You already touched this
`src/models/task.ts:9` | `TaskFilter` references `Task['status']` | Consumer — if you changed `status`, filter breaks
`src/api/index.ts` | Passes `Task` objects through `addTask`/`listTasks` | Indirect consumer via service layer
`src/cli/index.ts` | Passes `Task` objects through `addTask`/`getFormattedTasks` | Indirect consumer via service layer
`src/reports/summary.ts` | Passes `Task` objects to `formatTask` | Indirect consumer

## Blast Radius: `formatTask`

**4 direct consumers** (besides the definition itself):

| # | File | Line | Notes |
|---|------|------|-------|
| 1 | `src/cli/index.ts` | 10 | `formatTask(task)` on newly created task |
| 2 | `src/api/index.ts` | 15 | `tasks.map(formatTask)` in list endpoint |
| 3 | `src/services/task-service.ts` | 31 | `listTasks(filter).map(formatTask)` in `getFormattedTasks` |
| 4 | `src/reports/summary.ts` | 9 | `tasks.map(formatTask)` in summary report |

## Findings

Given the branch name `feat/42-add-priority`, if you're adding a `priority` field to the `Task` interface:

1. **`src/services/task-service.ts:7`** — `addTask()` constructs `Task` objects. It must populate the new `priority` field. **Likely missed.**

2. **`src/utils/format-task.ts:8-9`** — `formatTask` currently doesn't display priority. If the feature should show priority, this needs updating. You said you touched this — verify it includes the new field.

3. **`src/utils/format-task.ts:16-19`** — `derivePriority` hardcodes priority logic. If `Task` now has an explicit `priority` field, this function may be redundant or need updating.

4. **`src/reports/summary.ts`** — Uses `formatTask` to render tasks. If `formatTask` now shows priority, this is covered. But if the summary report should show priority *differently* (e.g., grouped by priority), it's **not covered**.

5. **`src/models/task.ts:9-11`** — `TaskFilter` only filters by `status` and `search`. If users should be able to filter by priority, **`TaskFilter` needs a `priority` field** and `listTasks` (line 18-28 of task-service) needs to handle it.

**Bottom line:** The most likely missed consumer is `addTask()` in `src/services/task-service.ts`, which constructs `Task` objects without the new field. `TaskFilter` is also a candidate if priority filtering is in scope. Run `npx tsc --noEmit` to let the compiler surface any type errors from the interface change.