@[user]:
Explore the codebase for issue #50: "Add priority field to tasks with custom rules".
Produce a structured exploration summary.

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
no git remotes found


Command exited with code 1

@[assistant]:


@[toolResult]:
prompt.md:1:Explore the codebase for issue #50: "Add priority field to tasks with custom rules".


@[assistant]:


@[toolResult]:
Explore the codebase for issue #50: "Add priority field to tasks with custom rules".
Produce a structured exploration summary.

@[assistant]:


@[toolResult]:
api
cli
models
reports
services
utils


@[assistant]:


@[toolResult]:
src/cli/index.ts
src/utils/format-task.ts
src/reports/summary.ts
src/services/task-service.ts
src/api/index.ts
src/models/task.ts


@[assistant]:


@[toolResult]:
export interface Task {
  readonly id: string;
  readonly title: string;
  readonly status: 'todo' | 'in_progress' | 'done';
  readonly createdAt: Date;
  readonly updatedAt: Date;
}

export interface TaskFilter {
  readonly status?: Task['status'];
  readonly search?: string;
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
**Issue Context**

- Repo prompt requests exploration for issue #50: “Add priority field to tasks with custom rules.”
- No detailed issue body or rules provided in repo history; assumptions must be clarified later (e.g., where custom rules live, whether configurable per task/user, etc.).

---

**What Exists**

1. **Models (`src/models/task.ts`)**
   - `Task` interface has `id`, `title`, `status`, `createdAt`, `updatedAt`—no `priority`.
   - `TaskFilter` allows filtering by `status` and `search` only.

2. **Service Layer (`src/services/task-service.ts`)**
   - In-memory `tasks` array of `Task`.
   - `addTask(title)` creates tasks with default `status = 'todo'`; no priority handling.
   - `listTasks(filter?)` supports `status` and text search filtering.
   - `getFormattedTasks(filter?)` returns formatted strings via `formatTask`.

3. **Utilities (`src/utils/format-task.ts`)**
   - `formatTask(task)` outputs a string with status icon, id, title—doesn’t display priority.
   - `derivePriority(task)` exists but is unused; returns `'high'` when `status === 'in_progress'`, else `'medium'`. Comment notes it’s a partial implementation lacking custom rules.

4. **Entry Points**
   - CLI (`src/cli/index.ts`): commands `add` and `list`, relies on service functions.
   - API (`src/api/index.ts`): GET `/tasks` returns formatted strings; POST `/tasks` creates tasks; no priority.
   - Reports (`src/reports/summary.ts`): uses `formatTask` for summaries.

---

**What Likely Needs to Change**

- **Models**: Extend `Task` with a `priority` field (type TBD, e.g., `'low' | 'medium' | 'high' | ...`), and possibly update `TaskFilter` to include priority-based queries.
- **Service Layer**:
  - Determine how priority is stored when tasks are created/updated (maybe derived versus stored).
  - If “custom rules” influence priority, need a configuration mechanism or rule engine (maybe external file/function).
  - Ensure `listTasks` and `getFormattedTasks` expose priority data (filtering, formatting).
- **Utilities**:
  - Finalize `derivePriority` to support “custom rules” (clarify source of rules: status-based? user-provided?). Ensure it’s used when tasks are created/updated or formatted.
  - Update `formatTask` to show priority.
- **Entry Points**:
  - CLI/API/report outputs should reflect priority.
  - If custom rules are user-configurable via CLI/API, interfaces need adjustments (not yet defined).
- **Data flow**: decide whether priority is persisted per task or computed on the fly.

---

**Consumers / Impacted Areas**

- `Task` interface consumers: services, CLI, API, reports, utilities. Every import of `Task` (`task-service`, `format-task`, API, CLI) will need updates.
- `formatTask` is used by:
  - CLI (`runCli`)
  - API (GET `/tasks`)
  - Reports (`generateSummary`)
  - `task-service.getFormattedTasks`
  All outputs will change when priority is added, requiring verification.

---

**Key Decisions Pending**

1. **Priority Representation**: string union? numeric? Should it support custom labels or ordering rules?
2. **Custom Rules Source**: Are rules static (hardcoded logic), configured via file/env, or user input? Need design direction.
3. **Storage vs Derivation**: Should priority be stored on the `Task` object or derived dynamically each time using `derivePriority` with rules?
4. **Filtering & Sorting**: Should list endpoints allow filtering or ordering by priority?
5. **Backward Compatibility**: Existing tasks (if persisted later) without priority—need migration/default behavior.

---

**Risks / Considerations**

- **Shared Utility Blast Radius**: `formatTask` changes affect CLI, API responses, and reports simultaneously.
- **Rule Complexity**: “Custom rules” may require a flexible engine; scope creep risk if not defined.
- **Testing Coverage**: Need tests covering priority derivation, formatting, and entry-point behavior.
- **State Management**: Currently tasks live in-memory; adding derived fields must remain consistent throughout lifecycle.

---

**Next Steps**

- Obtain full issue details/rules specification.
- Confirm whether to create a new branch/worktree (issue claiming).
- After requirements clarified, proceed to design (likely needed due to cross-cutting changes).