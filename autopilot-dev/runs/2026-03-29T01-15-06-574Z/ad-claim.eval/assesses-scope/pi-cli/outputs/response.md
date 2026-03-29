@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ad-claim/SKILL.md">
---
name: ad-claim
description: >-
  Use when starting work on a GitHub issue, setting up a development workspace,
  creating a worktree and branch, or when asked to "claim an issue", "start work on
  issue #N", "set up a branch for this", or "create a worktree". Handles issue
  assignment, worktree creation, branch setup, and draft PR creation.
---

# Claim

## Overview

Claim a GitHub issue and set up the full development workspace: worktree, branch, initial commit, and draft PR. This is the "I'm starting work" signal.

## Process

### Step 1: Read repo guidelines

Read ALL convention sources before writing any code:

- CLAUDE.md
- AGENTS.md
- CONTRIBUTING.md (if present)
- `.editorconfig`, `biome.json`, `.eslintrc`, pre-commit hook configs

Extract: worktree location, branch naming, commit format, lint rules, test commands.

If no conventions are specified, use these defaults:

| Convention | Default |
|---|---|
| Worktree location | `../<repo>.worktrees/<type>-<short-desc>` |
| Branch naming | `<type>/<issue-number>-<short-desc>` |
| Commit format | Conventional commits: `type(scope): description` |

### Step 2: Check issue state

- Verify the issue exists and is open
- Check it is unassigned or already assigned to self
- Read the full issue body — extract objective, constraints, acceptance signals, non-goals

### Step 3: Self-assign

```bash
gh issue edit <number> --add-assignee @me
```

### Step 4: Create worktree

```bash
git fetch origin
git worktree add <worktree-path> -b <branch-name> origin/main
```

### Step 5: Initial commit and draft PR

```bash
cd <worktree-path>
git commit --allow-empty -m "chore(<scope>): claim issue #<number>"
git push -u origin <branch-name>
gh pr create --draft --title "<type>(<scope>): <description>" --body "Closes #<number>"
```

### Step 6: Assess scope

After claiming, determine which phases to run next:

- **Trivial** (< 5 lines, docs, config): skip to ad-implement
- **Bug fix with clear root cause**: proceed to ad-explore, then ad-implement
- **Feature or complex change**: proceed to ad-explore → ad-design → ad-plan

## Hard Gates

- Must have an issue number — no untracked work
- Must read repo guidelines before any other action
- Must read the full issue body before assessing scope

## Multi-Repo Work

For changes spanning multiple repositories, the plugin defers to the repo's CLAUDE.md/AGENTS.md for coordination instructions. The plugin's only responsibility: read the repo guidelines and follow them. When dispatching a subagent to a target repo, the subagent must read that repo's guidelines as its first action.

</file>
[[User]]:
<file: path="plugins/autopilot-dev/skills/ad-claim/SKILL.md">

[[User]]:
I'm claiming this issue. Here is the full issue body:

**Issue #99: Fix typo in README.md — change 'taks' to 'tasks'**

The README.md file is in the repo root. Assess the scope of this change
and tell me which lifecycle phases are needed.


@[assistant]:


@[toolResult]:
---
name: ad-using-autopilot-dev
description: >-
  Use when starting any conversation or session to establish the agentic delivery
  lifecycle. Determines which phase skills to invoke and prevents rationalization
  ("this is too simple", "I'll just do this one thing first"). Skip this skill if
  dispatched as a subagent to execute a specific task.
---

# Using Autopilot Dev

## Overview

Entry point skill that establishes the phase-based delivery lifecycle and enforces skill invocation discipline.

## Subagent Stop

If you were dispatched as a subagent to execute a specific task, skip this skill entirely.

## Lifecycle

```
ad-claim → ad-explore → ad-design → ad-plan → ad-implement → ad-verify → ad-ship
```

| Phase | Skill | What Happens |
|---|---|---|
| Claim | ad-claim | Claim issue, create worktree + branch + draft PR |
| Explore | ad-explore | Understand the codebase and problem space |
| Design | ad-design | Brainstorm approaches, write approved spec |
| Plan | ad-plan | Convert spec into bite-sized implementation plan |
| Implement | ad-implement | TDD execution with subagent dispatch |
| Verify | ad-verify | E2E red/green testing, code review, blast radius check |
| Ship | ad-ship | Mark PR ready, merge, clean up worktree |

## Phase Skip Rules

Not every change needs every phase:

- **Trivial changes** (< 5 lines, docs-only, config-only): claim → implement → verify → ship
- **Bug fixes with clear root cause**: claim → explore → implement → verify → ship
- **Well-specified issues** (full spec in issue body): claim → explore → plan → implement → verify → ship

When in doubt, do not skip phases. Skipping design on a "simple" change that turns out to be complex is more expensive than spending 5 minutes on design.

## The 1% Rule

If there is even a 1% chance a phase skill applies to the current task, invoke it. Check for applicable skills BEFORE any response or action — including clarifying questions.

## Red Flags

These thoughts mean STOP — you are rationalizing your way out of a phase:

| Thought | Reality |
|---|---|
| "This is just a simple question" | Questions are tasks. Check the lifecycle. |
| "I need more context first" | That is what ad-explore does. |
| "Let me just write the code quickly" | That is what ad-implement does, with TDD. |
| "I can skip the design for this" | Every project needs a design, regardless of perceived simplicity. |
| "Tests are passing, we're done" | Unit tests ≠ verified. That is what ad-verify does. |
| "I'll clean up the PR later" | That is what ad-ship does, with blast radius checks. |
| "Let me explore the code first" | Use ad-explore — it has structured output. |
| "I know what needs to change" | Verify with ad-explore. Partial implementations may already exist. |

## Skill Priority

When multiple skills could apply:

1. **Process skills first** (ad-explore, ad-design) — determine HOW to approach
2. **Execution skills second** (ad-implement, ad-verify) — guide what to do

"Let's build X" → ad-explore first, then ad-design, then ad-plan.
"Fix this bug" → ad-explore first, then ad-implement.

## Artifact Locations

All plan and design artifacts live on the worktree branch, not in the main repo tree:

| Artifact | Location |
|---|---|
| Design specs | `.agents/plans/YYYY-MM-DD-<topic>-design.md` |
| Implementation plans | `.agents/plans/YYYY-MM-DD-<topic>-plan.md` |

## Configuration

The plugin reads conventions from the repo's CLAUDE.md, AGENTS.md, and contributing guides. Repo guidelines always override plugin defaults. The plugin provides workflow discipline — project-specific concerns belong in the project's guidelines.


@[assistant]:


@[toolResult]:
/home/christso/.agentv/workspaces/025d50ee-2f52-49de-8b5f-e7434319b4c4/shared


@[assistant]:


@[toolResult]:
# Task Tracker

A minimal task management CLI and API.

## Conventions

- **Worktree location:** `../task-tracker.worktrees/<branch-name>`
- **Branch naming:** `<type>/<issue-number>-<short-desc>`
- **Commit format:** Conventional commits — `type(scope): description`
- **Test command:** `npx vitest run`
- **Lint command:** `npx biome check .`
- **Build command:** `npx tsc --noEmit`

## Architecture

- `src/models/` — Data types and interfaces
- `src/services/` — Business logic
- `src/cli/` — CLI entry point
- `src/api/` — API entry point (Express)
- `src/utils/` — Shared utilities

Both CLI and API modes use the same service layer. Changes to services or models must be tested through both entry points.


@[assistant]:


@[toolResult]:
AGENTS.md
biome.json
CLAUDE.md
package.json
prompt.md
README.md
scripts
src
tsconfig.json


@[assistant]:


@[toolResult]:
**FIRST ACTION**: Read @AGENTS.md before any task.


@[assistant]:
This change is purely documentation: swapping one word (“taks”) for “tasks” in README.md. It’s a trivial doc-only edit (<5 lines), so no exploration, design, or planning phases are needed. Follow the fast path:

1. **ad-claim** – already in progress for issue #99.
2. **ad-implement** – edit README.md to fix the typo.
3. **ad-verify** – confirm the README renders correctly (no build/tests required since it’s docs-only, but double-check the file).
4. **ad-ship** – finalize the PR.

No additional lifecycle phases are warranted.