@[user]:
Explore the codebase to understand the impact of changing the formatTask function signature.
Find all files that use formatTask.

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
---
name: ad-explore
description: >-
  Use when starting work on a feature or bug fix to understand the codebase before
  proposing changes, when asked to "explore the code", "understand the problem",
  "find related code", or "check for existing implementations". Produces a structured
  summary of what exists, what needs to change, and key risks.
---

# Explore

## Overview

Structured codebase exploration before design. Understand the problem space, existing code, and prior art before proposing any changes.

## Process

### Step 1: Read the issue

Read the full issue body. Extract:

- Objective — what is the desired end state?
- Constraints — what must not change?
- Acceptance signals — how do we know it works?
- Non-goals — what is explicitly out of scope?

### Step 2: Check for existing implementations

Before proposing new code, search for prior art within the codebase:

```
Grep for related function names, types, and concepts.
```

Existing partial implementations change the design. A function that already does 80% of what you need is not a greenfield task — it is an extension.

### Step 3: Fan-out exploration

Use this tool priority (fastest first):

1. **Local filesystem** — Glob, Grep, Read for file patterns and code search
2. **Semantic search** — DeepWiki MCP or LSP for understanding dependency relationships
3. **Web** — WebFetch/WebSearch for external API docs or prior art

### Step 4: Find all consumers of shared interfaces

When the change touches a shared function, type, or interface:

```
Grep for all import sites and call sites.
```

Every consumer must be understood before design begins. A change to a shared interface with 3 consumers is a different problem than one with 30.

### Step 5: Check for duplicates

Search open and recently closed issues/PRs for related work. Avoid duplicating effort.

### Step 6: Produce summary

Output a structured summary to conversation context (not a file):

- **What exists** — relevant code, partial implementations, related utilities
- **What needs to change** — files to modify, new files needed
- **All consumers** — every call site of shared interfaces being modified
- **Key decisions** — design choices that need resolution
- **Risks** — backward compatibility, performance, blast radius

## Hard Gates

- Must understand existing code before proposing changes
- Must check for duplicate/related issues
- Must find all consumers of any shared interface being modified

## Resumability

Before re-exploring, check what is already in conversation context. Do not repeat work that has already been done in this session.


@[assistant]:


@[toolResult]:
/home/christso/.agentv/workspaces/f75a54be-d2a4-4f6e-bfb2-14eb2234625e/shared


@[assistant]:


@[toolResult]:
---
name: ad-explore
description: >-
  Use when starting work on a feature or bug fix to understand the codebase before
  proposing changes, when asked to "explore the code", "understand the problem",
  "find related code", or "check for existing implementations". Produces a structured
  summary of what exists, what needs to change, and key risks.
---

# Explore

## Overview

Structured codebase exploration before design. Understand the problem space, existing code, and prior art before proposing any changes.

## Process

### Step 1: Read the issue

Read the full issue body. Extract:

- Objective — what is the desired end state?
- Constraints — what must not change?
- Acceptance signals — how do we know it works?
- Non-goals — what is explicitly out of scope?

### Step 2: Check for existing implementations

Before proposing new code, search for prior art within the codebase:

```
Grep for related function names, types, and concepts.
```

Existing partial implementations change the design. A function that already does 80% of what you need is not a greenfield task — it is an extension.

### Step 3: Fan-out exploration

Use this tool priority (fastest first):

1. **Local filesystem** — Glob, Grep, Read for file patterns and code search
2. **Semantic search** — DeepWiki MCP or LSP for understanding dependency relationships
3. **Web** — WebFetch/WebSearch for external API docs or prior art

### Step 4: Find all consumers of shared interfaces

When the change touches a shared function, type, or interface:

```
Grep for all import sites and call sites.
```

Every consumer must be understood before design begins. A change to a shared interface with 3 consumers is a different problem than one with 30.

### Step 5: Check for duplicates

Search open and recently closed issues/PRs for related work. Avoid duplicating effort.

### Step 6: Produce summary

Output a structured summary to conversation context (not a file):

- **What exists** — relevant code, partial implementations, related utilities
- **What needs to change** — files to modify, new files needed
- **All consumers** — every call site of shared interfaces being modified
- **Key decisions** — design choices that need resolution
- **Risks** — backward compatibility, performance, blast radius

## Hard Gates

- Must understand existing code before proposing changes
- Must check for duplicate/related issues
- Must find all consumers of any shared interface being modified

## Resumability

Before re-exploring, check what is already in conversation context. Do not repeat work that has already been done in this session.


@[assistant]:


@[toolResult]:
src/api/index.ts:2:import { formatTask } from '../utils/format-task';
src/api/index.ts:15:    return { status: 200, body: tasks.map(formatTask) };
src/services/task-service.ts:2:import { formatTask } from '../utils/format-task';
src/services/task-service.ts:31:  return listTasks(filter).map(formatTask);
src/reports/summary.ts:2:import { formatTask } from '../utils/format-task';
src/reports/summary.ts:5: * Third consumer of formatTask — generates a summary report.
src/reports/summary.ts:9:  const lines = [`Task Summary (${tasks.length} total)`, '---', ...tasks.map(formatTask)];
src/utils/format-task.ts:7:export function formatTask(task: Task): string {
src/cli/index.ts:2:import { formatTask } from '../utils/format-task';
src/cli/index.ts:10:      return `Created: ${formatTask(task)}`;
prompt.md:1:Explore the codebase to understand the impact of changing the formatTask function signature.
prompt.md:2:Find all files that use formatTask.


@[assistant]:
**What exists**
- `src/utils/format-task.ts` defines `formatTask(task: Task): string`. It’s imported widely, so any signature change affects multiple layers (CLI, API, reports, services).
- Consumers:
  1. `src/cli/index.ts` – prints formatted tasks for CLI commands (creation responses and listings).
  2. `src/api/index.ts` – converts service task data into API response payloads.
  3. `src/services/task-service.ts` – service-level formatting when returning task lists.
  4. `src/reports/summary.ts` – builds a summary report string using formatted task lines.

**What needs to change**
- If `formatTask`’s signature changes, each of the four consumers above must be updated to match the new arguments/return type. Additional adjustments may be required in calling code (e.g., mapping logic, response shapes).

**All consumers**
- `src/cli/index.ts`
- `src/api/index.ts`
- `src/services/task-service.ts`
- `src/reports/summary.ts`

**Key decisions**
- Determine the new signature (extra parameters, different return type, async?). Decide whether callers should pass additional context or if formatting should remain centralized.
- Decide whether service/API layers should still rely on this shared formatter or diverge (e.g., API returning structured objects vs CLI/report strings).

**Risks**
- Backward compatibility: CLI/API outputs might change unexpectedly.
- Blast radius: shared utility touches both user-facing CLI and API responses; tests must cover both paths.
- Potential duplication if different layers begin reformatting independently—central contract must remain clear.