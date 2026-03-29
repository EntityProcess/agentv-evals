@[user]:
Explore the codebase for issue #50: "Add priority field to tasks with custom rules".
Understand what exists before proposing changes.

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
/home/christso/.agentv/workspaces/f75a54be-d2a4-4f6e-bfb2-14eb2234625e/shared


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
AGENTS.md:8:- **Branch naming:** `<type>/<issue-number>-<short-desc>`
prompt.md:1:Explore the codebase for issue #50: "Add priority field to tasks with custom rules".


@[assistant]:


@[toolResult]:
Explore the codebase for issue #50: "Add priority field to tasks with custom rules".
Understand what exists before proposing changes.

@[assistant]:


@[toolResult]:
**FIRST ACTION**: Read @AGENTS.md before any task.


@[assistant]:


@[toolResult]:
prompt.md:1:Explore the codebase for issue #50: "Add priority field to tasks with custom rules".


@[assistant]:


@[toolResult]:
scripts/setup.mjs:72:  execSync('git checkout -b feat/42-add-priority', {
src/utils/format-task.ts:13: * Partial implementation of priority derivation.
src/utils/format-task.ts:14: * Currently only handles basic cases — does not support custom priority rules.
prompt.md:1:Explore the codebase for issue #50: "Add priority field to tasks with custom rules".


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
api
cli
models
reports
services
utils


@[assistant]:


@[toolResult]:
task.ts


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
task-service.ts


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
index.ts


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
index.ts


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
src/utils/format-task.ts:16:export function derivePriority(task: Task): 'high' | 'medium' | 'low' {


@[assistant]:


@[toolResult]:
src/utils/format-task.ts:13: * Partial implementation of priority derivation.
src/utils/format-task.ts:14: * Currently only handles basic cases — does not support custom priority rules.


@[assistant]:


@[toolResult]:
## feat/42-add-priority
?? prompt.md


@[assistant]:


@[toolResult]:
summary.ts


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
src/utils/format-task.ts:13: * Partial implementation of priority derivation.
src/utils/format-task.ts:14: * Currently only handles basic cases — does not support custom priority rules.


@[assistant]:


@[toolResult]:
prompt.md:1:Explore the codebase for issue #50: "Add priority field to tasks with custom rules".


@[assistant]:


@[toolResult]:
undefined


@[assistant]:


@[toolResult]:
{
  "name": "task-tracker",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "tsc --noEmit",
    "test": "vitest run",
    "lint": "biome check ."
  },
  "devDependencies": {
    "typescript": "^5.8.0",
    "vitest": "^3.0.0",
    "@biomejs/biome": "^1.9.0"
  }
}


@[assistant]:
**Issue #50 context**

- Goal: add a `priority` field to tasks plus support for “custom rules” that determine the priority value.
- No detailed constraints/acceptance criteria were provided beyond adding the field and custom logic, so we must infer from current architecture.

---

### What already exists

1. **Task model** – `src/models/task.ts`
   - `Task` includes `id`, `title`, `status`, timestamps. No priority field yet.
   - `TaskFilter` supports `status` + text search only.

2. **Service layer** – `src/services/task-service.ts`
   - In-memory `tasks` array, `addTask`, `listTasks`, `getFormattedTasks`.
   - `addTask` hardcodes `status: 'todo'`, no priority handling.
   - `getFormattedTasks` delegates to `formatTask`. Any priority display logic must be centralized there.

3. **Shared formatter** – `src/utils/format-task.ts`
   - `formatTask` currently renders `[statusIcon] [id] title`.
   - `derivePriority` helper exists but returns `'high'` for `in_progress`, otherwise `'medium'`. Comments say it’s partial and lacks custom rules support. This is the only priority-related code today.
   - No code calls `derivePriority`.

4. **Entry points**
   - CLI (`src/cli/index.ts`) uses `addTask`, `getFormattedTasks`, directly calls `formatTask` when reporting “Created: …”.
   - API (`src/api/index.ts`) mirrors CLI behavior: GET `/tasks` returns formatted strings; POST `/tasks` returns the raw `Task`.
   - Reports module (`src/reports/summary.ts`) also uses `formatTask`. So any change to task shape/format affects CLI, API, and reports.

5. **Tooling**
   - Commands: `npx vitest run`, `npx biome check .`, `npx tsc --noEmit`.
   - Branch currently `feat/42-add-priority`, but issue at hand is #50 (branch/worktree might need realignment later).

---

### What needs to change (high-level)

- **Model**: `Task` interface will need a `priority` property (likely `'low' | 'medium' | 'high'`, or broader). All creations/updates must set it.
- **Service layer**: `addTask` must assign priority via rule engine; `listTasks`/filters may eventually filter by priority (depending on issue scope). Stored tasks must include priority persistently.
- **Formatter**: `formatTask` should probably include priority in output, and `derivePriority` needs full implementation with “custom rules.”
- **Custom rules mechanism**: Nothing exists yet. We must determine where rules live (config? environment? user input?). Exploration shows no current infra, so new module or config likely required.
- **Consumers**: CLI/API/report need to handle/display priority safely.

---

### Consumers of affected interfaces

- `Task` interface is imported/used in:
  - `src/models/task.ts` (definition)
  - `src/services/task-service.ts`
  - `src/utils/format-task.ts`
  - Potentially any future files; current consumers limited to service + formatter + API/CLI/report (since they import functions returning `Task` or formatted strings).

- `formatTask` is used by:
  - CLI (`src/cli/index.ts`)
  - API (`src/api/index.ts`)
  - Reports (`src/reports/summary.ts`)
  - Services (`getFormattedTasks`)

Any change to `Task` shape or `formatTask` output impacts all of the above.

---

### Key open decisions before implementation

1. **Priority data model**: Value domain (3 levels? numeric?). Should it be a dedicated type alias?
2. **Custom rule source**: Are rules hardcoded (e.g., based on status, keywords, creation date) or user-configurable? Need clarification from issue details or product owner.
3. **Persistence**: Since tasks are in-memory now, priority can be computed at creation and stored. Do rules re-evaluate dynamically (e.g., changing status updates priority) or only at creation time?
4. **Filtering/sorting**: Will CLI/API need ability to filter or sort by priority?
5. **Backward compatibility**: Existing tasks created before adding priority would need default assignment if persisted; currently there’s no persistence, so less of a concern unless future storage planned.

---

### Risks / blast radius

- **Shared formatter**: A change in `formatTask` affects all user-facing outputs. Need consistent formatting and tests across CLI/API/report.
- **Task type changes**: TypeScript compilation will fail until every `Task` instantiation and consumer handles the new field.
- **Rule complexity**: Without a spec, implementing “custom rules” could sprawl—need clear scope (e.g., priority derived from status + title keywords? user-provided config?).
- **Testing**: Need coverage for rule logic and all entry points to ensure consistent display/serialization.

---

Next step per process: await clarification/spec or proceed to design once requirements are clear.