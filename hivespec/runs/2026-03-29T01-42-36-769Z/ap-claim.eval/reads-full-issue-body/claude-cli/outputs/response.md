@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Here's what I understand from Issue #42:

**Objective:** Add an optional `dueDate` field to the `Task` interface.

**Acceptance criteria:**
- Tasks can be filtered by due date
- Existing tasks without due dates continue to work (backward compatibility)

**Non-goals:**
- No calendar UI
- No recurring tasks

**Scope assessment:** This is a **feature** touching models and services (filter logic), with both CLI and API entry points needing coverage per AGENTS.md. Recommended path: **ap-explore → ap-design → ap-plan**.

**Repo conventions extracted:**
| Convention | Value |
|---|---|
| Worktree location | `../task-tracker.worktrees/feat/42-add-due-date` |
| Branch name | `feat/42-add-due-date` |
| Commit format | `feat(task): ...` |
| Test command | `npx vitest run` |
| Lint command | `npx biome check .` |

Ready to proceed with Steps 3-5 (self-assign, create worktree, initial commit + draft PR) whenever you say go.