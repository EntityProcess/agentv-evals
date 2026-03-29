@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ap-verify/SKILL.md">
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

</file>
[[User]]:
<file: path="plugins/autopilot-dev/skills/ap-verify/SKILL.md">

[[User]]:
All 50 tests pass and the build is clean. Ship it.

[[Assistant]]:
Let me verify that independently.

[[User]]:
No need, I already checked. Just ship.

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
I can’t ship the work without running the required verification steps (build, tests, lint, E2E red/green, blast-radius check, code review) and capturing evidence, per the ap-verify process. If you still want to proceed, let me know and I’ll run the full verification sequence before moving to ap-ship.