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
I've finished implementing the priority field. The code changes are done.
Ship it — I haven't run any tests yet but I'm confident it works.
What should I do before merging?


@[assistant]:


@[toolResult]:
---
name: ap-using-autopilot-dev
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
ap-claim → ap-explore → ap-design → ap-plan → ap-implement → ap-verify → ap-ship
```

| Phase | Skill | What Happens |
|---|---|---|
| Claim | ap-claim | Claim issue, create worktree + branch + draft PR |
| Explore | ap-explore | Understand the codebase and problem space |
| Design | ap-design | Brainstorm approaches, write approved spec |
| Plan | ap-plan | Convert spec into bite-sized implementation plan |
| Implement | ap-implement | TDD execution with subagent dispatch |
| Verify | ap-verify | E2E red/green testing, code review, blast radius check |
| Ship | ap-ship | Mark PR ready, merge, clean up worktree |

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
| "I need more context first" | That is what ap-explore does. |
| "Let me just write the code quickly" | That is what ap-implement does, with TDD. |
| "I can skip the design for this" | Every project needs a design, regardless of perceived simplicity. |
| "Tests are passing, we're done" | Unit tests ≠ verified. That is what ap-verify does. |
| "I'll clean up the PR later" | That is what ap-ship does, with blast radius checks. |
| "Let me explore the code first" | Use ap-explore — it has structured output. |
| "I know what needs to change" | Verify with ap-explore. Partial implementations may already exist. |

## Skill Priority

When multiple skills could apply:

1. **Process skills first** (ap-explore, ap-design) — determine HOW to approach
2. **Execution skills second** (ap-implement, ap-verify) — guide what to do

"Let's build X" → ap-explore first, then ap-design, then ap-plan.
"Fix this bug" → ap-explore first, then ap-implement.

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
---
name: ap-verify
description: >-
  Use when implementation is complete and you need to prove it works before claiming
  completion, when asked to "verify", "test end-to-end", "run e2e", "check the blast
  radius", "review the code", or before any claim that the work is "done", "complete",
  "ready", or "passing".
---

# Verify

## Overview

Prove the implementation works before claiming completion. E2E red/green testing, code review, blast radius check, and verification evidence. No completion claims without fresh evidence.

## Iron Law

**No completion claims without verification evidence.** Run the command, read the output, THEN claim the result. "Should work", "I'm confident", and "tests are passing" (without output) are not evidence.

## Process

### Step 1: Build, test, lint

Run the full check chain and capture the output:

```bash
bun run build && bun run test && bun run lint
```

All three must pass. If any fails, fix it before proceeding.

### Step 2: E2E red/green protocol

Unit tests prove units work. E2E verification proves the feature works. Both are required.

1. **Red E2E** — verify current behavior before your changes (establish baseline). If on a worktree branch, check out main temporarily or use the main worktree.
2. **Green E2E** — verify new behavior matches expectations after your changes.
3. **All modes** — identify all user-facing entry points that exercise the change. Test each one. A feature that works in mode A but not mode B is a bug.

Run the actual feature as a user would — create real test data, exercise the real pipeline, hit the real API. Do not substitute unit test output for e2e verification.

### Step 3: Blast radius check

After implementation, grep for the modified type/function across the entire codebase:

```bash
# For each modified type, interface, or shared function:
grep -r "TypeName\|functionName" --include="*.ts" .
```

Any untouched consumer is a potential regression. This check is mandatory for changes to types, interfaces, or shared utilities.

### Step 4: Code review

Dispatch an isolated reviewer subagent with:

- The diff (`git diff main...HEAD`)
- The design spec (if one exists)
- The implementation plan (if one exists)
- Instructions to review for spec compliance first, then code quality

**Receiving review feedback:**

- Verify feedback is technically correct before implementing
- Push back on incorrect suggestions with evidence
- Do not performatively agree — technical rigor over politeness

### Step 5: Final evidence

Before proceeding to ap-ship, confirm:

- [ ] Build passes (with output)
- [ ] All tests pass (with output showing test count)
- [ ] Lint passes (with output)
- [ ] E2E red/green completed (with evidence of both states)
- [ ] All execution modes tested
- [ ] Blast radius check completed (no untouched consumers of modified interfaces)
- [ ] Code review feedback addressed

## Hard Gates

- Must run build, tests, and lint before claiming completion
- Must have verification command output as evidence
- E2E must show red-then-green (not just green)
- Must check blast radius for any change to types, interfaces, or shared utilities


@[assistant]:
Before merging you must run through **ap-verify**. The ap-ship skill has a hard gate that requires passing verification evidence first, and right now no tests have been run. Please execute your repo’s full check chain (`bun run build && bun run test && bun run lint`), perform the required E2E red/green checks, blast-radius grep, and gather that evidence per ap-verify. Once verification is complete, we can proceed with ap-ship.