@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ad-ship/SKILL.md">
---
name: ad-ship
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

Must have passing verification evidence from ad-verify before shipping. If no verification has been done in this session, stop and tell the user to run ad-verify first.

## Process

### Step 1: Final verification gate

Run the full check chain one last time:

```bash
bun run build && bun run test && bun run lint
```

All must pass with output as evidence. Do not skip this even if ad-verify ran recently — code may have changed since.

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
<file: path="plugins/autopilot-dev/skills/ad-ship/SKILL.md">

[[User]]:
The PR modifies the Task interface in src/models/task.ts and the formatTask
utility in src/utils/format-task.ts. All tests pass.
Before I mark this ready for review, what checks should I run?


@[assistant]:
Since you’re about to ship, follow the ad-ship process:

1. Final verification gate — rerun the full chain and capture output as evidence:
   ```bash
   bun run build && bun run test && bun run lint
   ```

2. Final blast-radius check — list the files touched by this PR and look for other consumers of the changed Task interface and formatTask utility:
   ```bash
   git diff main...HEAD --name-only
   rg "Task" --type-add 'ts:*.ts' --type ts
   rg "formatTask" --type-add 'ts:*.ts' --type ts
   ```
   (or equivalent grep commands). Confirm every affected consumer is updated.

3. Once those pass, you can proceed with push/PR steps (risk assessment, gh pr ready/merge, etc.).

That covers the required checks before marking the PR ready.