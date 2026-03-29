@[user]:
Explore the codebase for issue #50: "Add priority field to tasks with custom rules".
Understand what exists before proposing changes.

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
/home/christso/.agentv/workspaces/f3e3c3db-0a12-421c-9262-594b4a8d4fce/shared


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
prompt.md:1:Explore the codebase for issue #50: "Add priority field to tasks with custom rules".


@[assistant]:


@[toolResult]:
# Task Tracker

A minimal taks management CLI and API.

## Usage

```bash
# Add a task
task-tracker add "Buy groceries"

# List tasks
task-tracker list
```

## Development

```bash
npm install
npm run build
npm test
npm run lint
```


@[assistant]:


@[toolResult]:
Explore the codebase for issue #50: "Add priority field to tasks with custom rules".
Understand what exists before proposing changes.

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
src/utils/format-task.ts:13: * Partial implementation of priority derivation.
src/utils/format-task.ts:14: * Currently only handles basic cases — does not support custom priority rules.


@[assistant]:


@[toolResult]:
EISDIR: illegal operation on a directory, read

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
**Issue #50 context**

- **Objective**: introduce a priority field on tasks and support custom priority rules when deriving or displaying tasks.
- **Constraints / acceptance**: not explicitly provided; assume existing CLI/API behavior (add/list via shared service) must keep working while exposing the new priority information.

---

### What exists today

1. **Data model (`src/models/task.ts`)**
   - `Task` interface has `id`, `title`, `status`, `createdAt`, `updatedAt`.
   - `status` is `'todo' | 'in_progress' | 'done'`.
   - No priority field yet; filters operate on `status` and free-text search.

2. **Service layer (`src/services/task-service.ts`)**
   - In-memory `tasks: Task[]`.
   - `addTask(title)` builds `Task` objects with default `status: 'todo'`.
   - `listTasks(filter?)` filters by status/search.
   - `getFormattedTasks` maps `Task` via `formatTask`.
   - No persistence or priority handling; all Task creation flows through here.

3. **Shared formatting utilities (`src/utils/format-task.ts`)**
   - `formatTask(task)` turns a task into `"{icon} [id] title"`.
   - `derivePriority(task)` exists but is stubbed: returns `'high'` when `status === 'in_progress'`, otherwise `'medium'`.
   - Comment explicitly notes it “does not support custom priority rules,” hinting this file is the intended home for the new logic.

4. **Entry points / consumers**
   - **CLI (`src/cli/index.ts`)**: `add` command prints `Created: ${formatTask(task)}`; `list` shows formatted tasks.
   - **API (`src/api/index.ts`)**: `GET /tasks` responds with `tasks.map(formatTask)`; `POST /tasks` returns the raw `Task` object as JSON.
   - **Reports (`src/reports/summary.ts`)**: `generateSummary` uses `formatTask` to build report text.
   - These three modules are the direct consumers of `formatTask` (and indirectly any priority display logic) plus the shared service.

---

### What needs to change (high-level)

- **Model layer**: extend `Task` (and possibly `TaskFilter` if filtering by priority is required later) with a `priority` field or at least a way to derive it.
- **Service layer**: ensure `addTask` sets/derives the priority on creation or provides hooks for later derivation; update `listTasks`/`getFormattedTasks` if priority affects output or sorting.
- **Utility layer**: flesh out `derivePriority` with the “custom rules” the issue calls for, and decide how/where those rules are configured (constants, strategy functions, etc.). Update `formatTask` to include priority if needed.
- **Entry points**: CLI/API/report outputs must reflect priority additions while remaining backward-compatible (e.g., API responses now include a `priority` field on returned tasks).
- **Tests**: none currently exist, so new coverage (likely via Vitest) will be required around services/utilities to lock in the new behavior.

---

### Consumers / blast radius

- `Task` type is imported by `task-service.ts`, `format-task.ts`, and any future modules using tasks. Changing the interface impacts all creators/readers (CLI/API/report transitively).
- `formatTask` is consumed by CLI, API, and reports; any signature/output change affects all user-facing channels simultaneously.
- `derivePriority` isn’t used elsewhere yet, but once wired into services or formatters it will affect every display path.

---

### Key decisions to resolve

1. **Priority representation**: is it a string union (`'low' | 'medium' | 'high'`), numeric weight, or more elaborate structure? Should it live directly on the `Task` object or be computed on the fly?
2. **Custom rule configuration**: Where do rules come from? Hard-coded heuristics (status-based, title keywords, age), or user-provided config? Do rules need to be extensible?
3. **Data lifecycle**: Should priority be persisted alongside tasks (so API `POST /tasks` returns `priority`), or purely derived when formatting?
4. **Backward compatibility**: how to handle existing tasks (in-memory here, but in a real system there may already be stored tasks without priority). Need sensible defaults.

---

### Risks

- **Interface changes**: Adding mandatory fields to `Task` or altering `formatTask` output can break existing consumers/tests; ensure all call sites updated and document API changes.
- **Rule complexity**: “Custom rules” could imply user-defined logic; unclear requirements may lead to over/under-engineering. Need clarification before coding.
- **Shared utility coupling**: Since CLI/API/reports all rely on `formatTask`, mistakes in priority formatting affect every user channel simultaneously.
- **Testing gap**: No existing test suite; new logic must include fresh tests to prevent regressions.

This analysis should equip us to either request clarification on custom rule expectations or move into design/planning with the impacted files already identified.