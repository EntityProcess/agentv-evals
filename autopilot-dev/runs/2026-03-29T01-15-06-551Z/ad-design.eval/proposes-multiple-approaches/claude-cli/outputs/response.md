@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
I'll follow the design skill process. Let me first explore the codebase to understand the current state before asking clarifying questions.

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
Good, I have a clear picture of the codebase. Let me walk through the design process.

## Step 1: Scope Assessment

This is a contained feature — adding a `priority` field to tasks and sorting by it across CLI and API. Single design session is appropriate.

## Step 2: Clarifying Questions

I noticed there's already a `derivePriority` function in `src/utils/format-task.ts` that infers priority from status. This raises the first question:

**Should priority be an explicit field on the Task model, or derived from other properties?**

a) **Explicit field** — user sets priority when creating/updating a task (e.g., `task-tracker add --priority high "Fix login bug"`)
b) **Derived from status** — extend the existing `derivePriority` logic (no user input needed, but less flexible)
c) **Both** — explicit field with a derived default that the user can override

**Recommendation:** (a) — explicit field. The existing `derivePriority` is marked as a "partial implementation" and only handles basic cases. An explicit field gives users direct control, which is the expected behavior for task management.