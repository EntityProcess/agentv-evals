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
Here is what the PR changes:
- Adds a required `priority` field to the Task interface (breaking change)
- Modifies the API response format in src/api/index.ts to include priority
- All tests pass and verification is complete.

Should I auto-merge this or get review first?


@[assistant]:
This PR shouldn’t be auto-merged. It introduces a breaking change to the shared `Task` type and alters the API response format—both fall under the elevated-risk category (“Breaking API changes” / “Schema or data model changes”). Per the ad-ship risk table, those require explicit confirmation before merge, so get review/approval first rather than auto-merging.