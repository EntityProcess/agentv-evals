@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Using the **ad-claim** skill to claim issue #99. Starting with Step 1 — reading repo guidelines.

I already have CLAUDE.md and AGENTS.md from context. Let me check for additional convention sources.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Good. Here's what I've extracted from repo guidelines:

| Convention | Value |
|---|---|
| Worktree location | `../task-tracker.worktrees/<branch-name>` |
| Branch naming | `<type>/<issue-number>-<short-desc>` |
| Commit format | Conventional commits |
| Lint | `npx biome check .` |
| Test | `npx vitest run` |
| Build | `npx tsc --noEmit` |

**Step 2: Check issue state** — You've provided the full issue body:
- **Objective:** Fix typo in README.md — change 'taks' to 'tasks'
- **Constraints:** Single file (README.md), single word change
- **Acceptance:** The typo is corrected
- **Non-goals:** None stated

**Step 6: Assess scope**

This is **Trivial** — a single-word typo fix in a documentation file (< 5 lines, docs category). The lifecycle phases needed are:

1. **ad-claim** (current) — assign, worktree, branch, draft PR
2. **ad-implement** — make the one-line fix directly (skip explore, design, and plan)
3. **ad-verify** — confirm the change is correct
4. **ad-ship** — mark PR ready for review

No need for ad-explore, ad-design, or ad-plan — the root cause and fix are self-evident.

Want me to proceed with Steps 3-5 (self-assign, create worktree, initial commit, and draft PR)?