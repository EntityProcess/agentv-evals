@[user]:
Explore the codebase to understand the impact of changing the formatTask function signature.
Find all files that use formatTask.

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
name: ap-explore
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
prompt.md:1:Explore the codebase to understand the impact of changing the formatTask function signature.
prompt.md:2:Find all files that use formatTask.
src/utils/format-task.ts:7:export function formatTask(task: Task): string {
src/api/index.ts:2:import { formatTask } from '../utils/format-task';
src/api/index.ts:15:    return { status: 200, body: tasks.map(formatTask) };
src/cli/index.ts:2:import { formatTask } from '../utils/format-task';
src/cli/index.ts:10:      return `Created: ${formatTask(task)}`;
src/services/task-service.ts:2:import { formatTask } from '../utils/format-task';
src/services/task-service.ts:31:  return listTasks(filter).map(formatTask);
src/reports/summary.ts:2:import { formatTask } from '../utils/format-task';
src/reports/summary.ts:5: * Third consumer of formatTask — generates a summary report.
src/reports/summary.ts:9:  const lines = [`Task Summary (${tasks.length} total)`, '---', ...tasks.map(formatTask)];


@[assistant]:


@[toolResult]:
Explore the codebase to understand the impact of changing the formatTask function signature.
Find all files that use formatTask.

@[assistant]:


@[toolResult]:
import type { Task } from '../models/task';

/**
 * Shared utility used by CLI, API, and reports.
 * Format a task for display output.
 */
export function formatTask(task: Task): string {
  const statusIcon = task.status === 'done' ? '✓' : task.status === 'in_progress' ? '→' : '○';
  return `${statusIcon} [${task.id}] ${task.title}`;
}

/**
 * Partial implementation of priority derivation.
 * Currently only handles basic cases — does not support custom priority rules.
 */
export function derivePriority(task: Task): 'high' | 'medium' | 'low' {
  if (task.status === 'in_progress') return 'high';
  return 'medium';
}


@[assistant]:


@[toolResult]:
import { addTask, listTasks } from '../services/task-service';
import { formatTask } from '../utils/format-task';

/**
 * API handler — mirrors CLI functionality over HTTP.
 * Both entry points share the same service layer.
 */
export function handleRequest(
  method: string,
  path: string,
  body?: Record<string, unknown>,
): { status: number; body: unknown } {
  if (method === 'GET' && path === '/tasks') {
    const tasks = listTasks();
    return { status: 200, body: tasks.map(formatTask) };
  }

  if (method === 'POST' && path === '/tasks') {
    const title = body?.title as string;
    if (!title) return { status: 400, body: { error: 'title is required' } };
    const task = addTask(title);
    return { status: 201, body: task };
  }

  return { status: 404, body: { error: 'not found' } };
}


@[assistant]:


@[toolResult]:
import { addTask, getFormattedTasks } from '../services/task-service';
import { formatTask } from '../utils/format-task';

export function runCli(args: string[]): string {
  const [command, ...rest] = args;

  switch (command) {
    case 'add': {
      const task = addTask(rest.join(' '));
      return `Created: ${formatTask(task)}`;
    }
    case 'list':
      return getFormattedTasks().join('\n') || 'No tasks found.';
    default:
      return 'Usage: task-tracker [add|list] [args...]';
  }
}


@[assistant]:


@[toolResult]:
import type { Task, TaskFilter } from '../models/task';
import { formatTask } from '../utils/format-task';

const tasks: Task[] = [];

export function addTask(title: string): Task {
  const task: Task = {
    id: `task-${tasks.length + 1}`,
    title,
    status: 'todo',
    createdAt: new Date(),
    updatedAt: new Date(),
  };
  tasks.push(task);
  return task;
}

export function listTasks(filter?: TaskFilter): Task[] {
  let result = [...tasks];
  if (filter?.status) {
    result = result.filter((t) => t.status === filter.status);
  }
  if (filter?.search) {
    const q = filter.search.toLowerCase();
    result = result.filter((t) => t.title.toLowerCase().includes(q));
  }
  return result;
}

export function getFormattedTasks(filter?: TaskFilter): string[] {
  return listTasks(filter).map(formatTask);
}


@[assistant]:


@[toolResult]:
import { listTasks } from '../services/task-service';
import { formatTask } from '../utils/format-task';

/**
 * Third consumer of formatTask — generates a summary report.
 */
export function generateSummary(): string {
  const tasks = listTasks();
  const lines = [`Task Summary (${tasks.length} total)`, '---', ...tasks.map(formatTask)];
  return lines.join('\n');
}


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
Exploration summary (ap-explore):

- **What exists**:  
  - `src/utils/format-task.ts` exports `formatTask(task: Task): string`, shared by CLI/API/reports.  
  - Consumers:
    - `src/api/index.ts`: `GET /tasks` returns `tasks.map(formatTask)`.  
    - `src/cli/index.ts`: `add` command prints `Created: ${formatTask(task)}`; `list` indirectly via `getFormattedTasks`.  
    - `src/services/task-service.ts`: `getFormattedTasks()` maps `formatTask` over `listTasks`.  
    - `src/reports/summary.ts`: `generateSummary()` builds report lines using `formatTask`.  
- **What needs to change**: Any signature change to `formatTask` affects all four consumers plus any utilities that re-export or wrap `getFormattedTasks`. Need to update imports/usages and adjust `getFormattedTasks` if its return/output relies on the old signature.
- **All consumers**: Listed above (`src/api/index.ts`, `src/cli/index.ts`, `src/services/task-service.ts`, `src/reports/summary.ts`). No other matches per repo search.
- **Key decisions ahead**: New signature design (extra params? different return type?), whether to update downstream functions like `getFormattedTasks` or create adapters to maintain backward compatibility.
- **Risks**: CLI/API output regressions, report formatting changes, possible need to adjust tests covering formatted output. Since formatTask feeds both user-facing outputs and internal report generation, changing its return type or parameters has wide blast radius.