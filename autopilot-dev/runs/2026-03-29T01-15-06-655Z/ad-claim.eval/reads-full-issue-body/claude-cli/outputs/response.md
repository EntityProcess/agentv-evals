@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Here's what I understand from issue #42:

**Issue:** #42 — Add due date field to tasks

**Objective:** Add an optional `dueDate` field to the `Task` interface.

**Acceptance criteria:**
- Tasks can be filtered by due date
- Existing tasks without due dates continue to work (backward compatibility)

**Non-goals:**
- No calendar UI
- No recurring tasks

**Scope assessment:** This is a **feature** touching models, services, and both entry points (CLI + API per AGENTS.md). It will need:
- Model change in `src/models/` (add optional `dueDate` to Task)
- Service logic in `src/services/` (filtering by due date)
- CLI support in `src/cli/`
- API support in `src/api/`

**Recommended path:** ad-explore → ad-design → ad-plan → ad-implement

**Note:** The current branch is `feat/42-add-priority` — the branch name references "priority" rather than "due-date". Want me to proceed with claiming (self-assign, worktree, draft PR) using the correct naming `feat/42-add-due-date`, or continue on this existing branch?