@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Here's the impact analysis for `formatTask`:

**Definition:** `src/utils/format-task.ts:7` — `formatTask(task: Task): string`

**4 consumers across 3 layers:**

| File | Line | Usage pattern |
|---|---|---|
| `src/cli/index.ts` | 10 | `formatTask(task)` — single task formatting |
| `src/api/index.ts` | 15 | `tasks.map(formatTask)` — passed as callback to `.map()` |
| `src/services/task-service.ts` | 31 | `listTasks(filter).map(formatTask)` — passed as callback to `.map()` |
| `src/reports/summary.ts` | 9 | `tasks.map(formatTask)` — passed as callback to `.map()` |

**Key risk:** Three of the four call sites pass `formatTask` directly as a `.map()` callback (point-free style). If you add a second parameter to the signature, `.map()` will silently pass the array index as that argument, which could cause subtle bugs. Those three sites would need to be changed to `tasks.map(t => formatTask(t, ...))` with an explicit wrapper.

The CLI call site (`formatTask(task)`) is a direct call and is lower risk — it just needs the new argument added.