@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ap-ship/SKILL.md">
---
name: ap-ship
description: >-
  Use when implementation and verification are complete and you need to integrate the
  work, when asked to "ship it", "merge the PR", "mark ready for review", "clean up
  the branch", or "finish this work". Handles final verification, PR management, merge,
  and worktree cleanup.
---

# Ship

## Overview

Complete the development branch and integrate the work. Final verification gate, PR management, merge, and worktree cleanup.

## Hard Gate

Must have passing verification evidence from ap-verify before shipping. If no verification has been done in this session, stop and tell the user to run ap-verify first.

## Process

### Step 1: Final verification gate

Run the full check chain one last time:

```bash
bun run build && bun run test && bun run lint
```

All must pass with output as evidence. Do not skip this even if ap-verify ran recently — code may have changed since.

### Step 2: Final blast radius check

Before marking the PR ready, grep for the primary types and functions changed in this PR across the entire codebase:

```bash
git diff main...HEAD --name-only  # Files changed in this PR
# For each modified shared type/function:
grep -r "TypeName\|functionName" --include="*.ts" .
```

Any consumer not touched by this PR is a potential miss. This is the last line of defense.

### Step 3: Push and mark ready

```bash
git push
gh pr ready <number>
```

### Step 4: Risk classification

Assess the PR for merge risk:

| Auto-merge (low risk) | Confirm before merge (elevated risk) |
|---|---|
| Documentation changes | Breaking API changes |
| Config file updates | Feature deletion |
| Additive features (new files only) | Schema or data model changes |
| Isolated bug fixes | Security-sensitive changes |
| Style/formatting | Cross-repo coordination |
| Test additions | Changes to shared types with many consumers |

### Step 5: Merge

For auto-merge candidates, squash merge:

```bash
gh pr merge <number> --squash
```

For elevated risk, present the risk assessment and wait for explicit user confirmation before merging.

### Step 6: Worktree cleanup

After merge:

```bash
git worktree remove <worktree-path>
cd <main-worktree>
git pull origin main
```

### Step 7: Close tracking

- Verify the linked issue was closed by the merge
- For multi-repo work: link related PRs, update tracking issues

## Cross-Repo Work

For changes spanning multiple repositories, follow the coordination instructions in the repo's CLAUDE.md/AGENTS.md. The plugin's additions:

- Link related PRs across repos in PR descriptions
- Cross-repo PRs require explicit user confirmation before merging
- Update tracking issues with links to all related PRs

</file>
[[User]]:
<file: path="plugins/autopilot-dev/skills/ap-ship/SKILL.md">

[[User]]:
The PR modifies the Task interface in src/models/task.ts and the formatTask
utility in src/utils/format-task.ts. All tests pass.
Before I mark this ready for review, what checks should I run?


@[assistant]:
Since you’re at the “ready for review” gate, follow the ap-ship checklist before proceeding:

1. **Final verification gate** — run the full check chain, even if tests already passed earlier:
   ```bash
   bun run build && bun run test && bun run lint
   ```
   Keep the output as evidence.

2. **Blast radius check** — from the PR’s worktree:
   ```bash
   git diff main...HEAD --name-only
   ```
   Identify the shared types/functions you touched (here, `Task` in `src/models/task.ts` and anything exported by `formatTask`). For each, grep the repo to ensure all consumers are either updated or intentionally unchanged:
   ```bash
   grep -r "Task" --include="*.ts" .
   grep -r "formatTask" --include="*.ts" .
   ```
   Review any untouched consumers for required updates.

Once those steps are clean, you can proceed to push, mark ready, and continue with the rest of the ap-ship flow.