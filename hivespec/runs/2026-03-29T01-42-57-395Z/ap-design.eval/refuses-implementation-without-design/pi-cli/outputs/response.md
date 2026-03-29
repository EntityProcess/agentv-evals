@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ap-design/SKILL.md">
---
name: ap-design
description: >-
  Use when a feature or change needs a design before implementation, when the scope
  is non-trivial, when asked to "brainstorm", "design this", "write a spec", "propose
  approaches", or when multiple valid implementation strategies exist. Produces an
  approved design spec through collaborative dialogue.
---

# Design

## Overview

Turn exploration findings into a validated design through collaborative dialogue. Prevents implementation without an approved spec.

## Process

### Step 1: Assess scope

If the change is too large for a single design session, decompose into sub-projects first. Each sub-project gets its own design cycle.

### Step 2: Clarifying questions

Ask questions one at a time. Prefer multiple choice over open-ended:

```
Should this support both CLI and API modes?
a) CLI only (simpler, covers current use cases)
b) Both CLI and API (more flexible, more work)
c) API only (if CLI is being deprecated)

Recommendation: (b) — the API mode is needed for pipeline integration.
```

### Step 3: Propose approaches

Present 2-3 approaches with trade-offs and a recommendation:

| Approach | Pros | Cons |
|---|---|---|
| A: Extend existing | Low risk, fast | Limited flexibility |
| B: New abstraction | Clean design | More code to maintain |
| C: External library | Battle-tested | New dependency |

**Recommendation:** Approach A — explain why.

### Step 4: Section-by-section approval

Present the design in sections. Get user approval after each before proceeding:

1. Data model / types
2. API / interface changes
3. Implementation approach
4. Migration / backward compatibility

### Step 5: Write spec

Save the approved design to the worktree branch:

```
.agents/plans/YYYY-MM-DD-<topic>-design.md
```

### Step 6: Self-review

Before presenting the spec to the user, check:

- No placeholders ("TBD", "to be determined", "similar to above")
- No ambiguous language ("might", "could potentially", "if needed")
- Consistent naming throughout
- All decisions from the dialogue are captured
- Acceptance signals are concrete and testable

### Step 7: User reviews written spec

The user reviews the written spec file. Design is not approved until the user confirms.

## Hard Gate

No implementation until design is approved. Every project, regardless of perceived simplicity. The thought "this is too simple to need a design" is the strongest signal that a design is needed.

## Visual Companions

For UI changes, include mockups or wireframes (ASCII, Mermaid, or browser-based) alongside the spec. Visual designs prevent misalignment more effectively than prose.

</file>
[[User]]:
<file: path="plugins/autopilot-dev/skills/ap-design/SKILL.md">

[[User]]:
Add a priority system to tasks. High priority tasks should appear first in listings,
support custom priority rules, and work across both CLI and API modes.
Just go ahead and implement it.


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
---
name: ap-claim
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

- **Trivial** (< 5 lines, docs, config): skip to ap-implement
- **Bug fix with clear root cause**: proceed to ap-explore, then ap-implement
- **Feature or complex change**: proceed to ap-explore → ap-design → ap-plan

## Hard Gates

- Must have an issue number — no untracked work
- Must read repo guidelines before any other action
- Must read the full issue body before assessing scope

## Multi-Repo Work

For changes spanning multiple repositories, the plugin defers to the repo's CLAUDE.md/AGENTS.md for coordination instructions. The plugin's only responsibility: read the repo guidelines and follow them. When dispatching a subagent to a target repo, the subagent must read that repo's guidelines as its first action.


@[assistant]:
To keep us inside the required Autopilot Dev flow, I can’t jump straight into implementation. We need to:

1. **Claim an issue** (ap-claim) — I’ll need the GitHub issue number so I can create the worktree/branch and open the draft PR.
2. **Explore + Design** — this change definitely needs a spec (ap-design), so we’ll capture the priority model, CLI/API surfaces, and custom rule handling before writing code.

Could you share the relevant issue number (or confirm it’s okay to create one)? Once I have that, I’ll kick off the claim phase and we can move through design toward implementation.