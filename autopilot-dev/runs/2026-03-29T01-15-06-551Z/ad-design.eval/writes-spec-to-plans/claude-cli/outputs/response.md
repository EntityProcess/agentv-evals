@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Let me explore the codebase to understand the current task model before writing the spec.

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


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Now I have a clear picture. Let me read the existing `derivePriority` function since it's directly relevant.

@[assistant]:


@[assistant]:
Good. The existing `derivePriority` is a partial, status-based heuristic — Approach B replaces this with explicit user-set priority as a first-class field. Let me check for the plans directory and then write the spec.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
The spec is written to `.agents/plans/2026-03-29-task-priority-design.md`. Here's a summary of what it covers:

**Approach B: Explicit priority as a first-class field**

- **Data model:** New `Priority` type (`'low' | 'medium' | 'high' | 'urgent'`) added to `Task` interface and `TaskFilter`
- **Service:** `addTask` accepts optional priority (defaults to `'medium'`), `listTasks` supports priority filtering, new `updateTask` function
- **CLI:** `--priority` flag on `add` and `list`, new `update` command
- **API:** `priority` field on POST /tasks, new PATCH /tasks/:id route, query param filtering on GET /tasks
- **Utilities:** `formatTask` displays non-medium priorities in parens; `derivePriority` is removed
- **Backward compatible:** All existing calls work unchanged (priority defaults to medium)
- **15 concrete acceptance criteria** covering service, CLI, and API

Please review the spec and let me know if anything needs adjustment before we move to implementation.