@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Using the **ad-claim** skill to set up the workspace.

**Step 1: Reading repo guidelines**

I already have CLAUDE.md and AGENTS.md from context. Let me check for additional config files.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Good. Here's a summary of the conventions extracted:

| Convention | Value |
|---|---|
| Worktree location | `../task-tracker.worktrees/<branch-name>` |
| Branch naming | `<type>/<issue-number>-<short-desc>` |
| Commit format | Conventional commits |
| Test command | `npx vitest run` |
| Lint command | `npx biome check .` |
| Build command | `npx tsc --noEmit` |
| Formatter | Biome, spaces, indent 2 |

**Step 2-3: Skipping** — you said no GitHub needed, and the branch `feat/42-add-priority` already exists.

Note: The current branch is `feat/42-add-priority` which references "priority", not "due date". I'll proceed with this branch as-is since you said it's already created.

**Step 4: Creating worktree**

@[assistant]:


@[assistant]:
The current working directory *is* already checked out on `feat/42-add-priority` — so we're effectively already in the worktree. No separate worktree needed.

**Step 5: Skipping draft PR** — per your instructions, no GitHub.

**Step 6: Scope assessment**

Adding a due date field to tasks touches the data model, service layer, and likely both CLI and API entry points. Let me quickly check the current model to confirm.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
**Scope assessment: Feature (medium complexity)**

Adding a `dueDate` field will touch:
- **Model** (`src/models/task.ts`) — new optional `dueDate` field
- **Service** (`src/services/task-service.ts`) — accept/store due date on create/update
- **CLI** (`src/cli/index.ts`) — `--due-date` flag
- **API** (`src/api/index.ts`) — accept due date in request body
- **Formatter** (`src/utils/format-task.ts`) — display due date
- **Reports** (`src/reports/summary.ts`) — possibly include due date info
- Tests for all the above

This is a **feature with multiple touchpoints** — the recommended path is: **ad-explore → ad-design → ad-plan**.

---

Workspace is ready on branch `feat/42-add-priority`. Want me to proceed with **exploration** of the codebase to understand the current service/CLI/API patterns before designing the due date feature?