@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Following the ap-claim skill. Let me read the remaining convention sources and check the repo structure.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Good. Let me read the key source files to understand the current task model.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Repo guidelines extracted. Here's the workspace summary:

**Conventions found:**
- **Worktree location:** `../task-tracker.worktrees/<branch-name>`
- **Branch naming:** `<type>/<issue-number>-<short-desc>` — current branch `feat/42-add-priority` follows this
- **Commit format:** Conventional commits (`type(scope): description`)
- **Test:** `npx vitest run` | **Lint:** `npx biome check .` | **Build:** `npx tsc --noEmit`
- **Formatter:** Biome, 2-space indent, spaces

**Current state:**
- Already on branch `feat/42-add-priority`, clean working tree
- No worktree needed — you're already working in the repo directly

**Codebase snapshot:**
- `src/models/task.ts` — `Task` interface with `id`, `title`, `status`, `createdAt`, `updatedAt`; `TaskFilter` with `status` and `search`
- `src/services/task-service.ts` — `addTask`, `listTasks`, `getFormattedTasks` (in-memory store)
- `src/cli/index.ts`, `src/api/index.ts` — dual entry points sharing the service layer
- `src/utils/format-task.ts`, `src/reports/summary.ts` — formatting/reporting

**Scope assessment:** This is a **feature** (adding a `dueDate` field to the `Task` model). It touches models, services, CLI, API, and formatting — multiple layers. Per the skill, the recommended next phases are: **ap-explore → ap-design → ap-plan**.

Ready to proceed. Want me to kick off the explore phase to map all the touch points for the due date field?