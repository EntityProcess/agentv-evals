@[assistant]:


@[assistant]:
I'll invoke the exploration skill and start gathering context in parallel.

@[assistant]:


@[assistant]:


@[assistant]:
Using the **ad-explore** skill to systematically understand the codebase before proposing changes.

**Step 1: Read the issue + Step 2-3: Explore codebase in parallel**

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
No remote configured, so I can't fetch the issue from GitHub. Let me check for issue context in the repo and explore the full codebase.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Now let me check for any issue description files or `.pi` directory for context on the issue.

@[assistant]:


@[assistant]:
No issue file or remote — the issue context comes from the user's request title: **"Add priority field to tasks with custom rules"**. Here's the exploration summary:

---

## Exploration Summary: Issue #50 — Add priority field to tasks

### What exists

| File | Role |
|------|------|
| `src/models/task.ts` | `Task` interface (id, title, status, createdAt, updatedAt) — **no priority field yet** |
| `src/models/task.ts` | `TaskFilter` interface (status, search) — **no priority filter** |
| `src/services/task-service.ts` | `addTask(title)` and `listTasks(filter)` — no priority awareness |
| `src/utils/format-task.ts` | `formatTask()` — display helper, no priority shown |
| `src/utils/format-task.ts:16-19` | **`derivePriority()` — partial implementation exists!** Maps `in_progress → high`, everything else → `medium`. Comment says "does not support custom priority rules" |
| `src/cli/index.ts` | CLI entry point — `add` and `list` commands |
| `src/api/index.ts` | API entry point — `GET /tasks`, `POST /tasks` |
| `src/reports/summary.ts` | Summary report generator |

### Prior art: `derivePriority()`

There's already a stub at `src/utils/format-task.ts:16-19` that derives priority from status. It's **never called anywhere** — zero consumers. This is the starting point, not greenfield.

### All consumers of shared interfaces

**`Task` interface** — used in every file (6 files). Adding a `priority` field touches all of them.

**`formatTask()`** — 3 consumers:
1. `src/cli/index.ts:10` — CLI add command
2. `src/api/index.ts:14` — API list endpoint
3. `src/reports/summary.ts:10` — summary report

**`TaskFilter`** — 2 consumers:
1. `src/services/task-service.ts:18` — `listTasks()`
2. `src/services/task-service.ts:30` — `getFormattedTasks()`

### What needs to change

1. **`Task` interface** — add `priority: 'high' | 'medium' | 'low'` field
2. **`addTask()`** — accept optional priority param, default via `derivePriority()` or a custom rule system
3. **`derivePriority()`** — extend to support "custom rules" (the issue title's key requirement)
4. **`formatTask()`** — display priority in output
5. **`TaskFilter`** — optionally add priority filtering
6. **CLI** — accept `--priority` flag on `add` command
7. **API** — accept `priority` in POST body, optionally filter on GET
8. **Tests** — none exist yet; should be added

### Key decisions needed

1. **What are "custom priority rules"?** Keyword-based? User-defined callbacks? Config-driven? The stub only uses status — the design needs to define the rule system.
2. **Should priority be stored or derived?** Currently `derivePriority()` computes it from status. Should `Task` store an explicit priority that can be overridden, with `derivePriority()` as the default?
3. **Should priority be mutable after creation?** (e.g., re-derive when status changes)
4. **Filter by priority?** Extend `TaskFilter` or keep it simple for now?

### Risks

- **Blast radius**: `Task` interface is in every file — all 6 `.ts` files will need updates
- **No tests exist**: changes have no safety net; tests should be added alongside
- **`derivePriority` is unused**: need to decide whether to integrate it or replace it

---

This gives us a solid map. The next step would be **design** — deciding the rule system and whether priority is stored vs. derived. Want me to proceed to the design phase?