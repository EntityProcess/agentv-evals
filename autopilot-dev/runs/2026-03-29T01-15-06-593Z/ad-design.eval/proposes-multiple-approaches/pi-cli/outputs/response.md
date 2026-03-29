@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ad-design/SKILL.md">
---
name: ad-design
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
<file: path="plugins/autopilot-dev/skills/ad-design/SKILL.md">

[[User]]:
Design a solution for adding task priorities.
Tasks should be sortable by priority in both CLI and API modes.


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
---
name: ad-design
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
@[User]:
<file: path="plugins/autopilot-dev/skills/ad-design/SKILL.md">

@[User]:
Design a solution for adding task priorities.
Tasks should be sortable by priority in both CLI and API modes.


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
format-task.ts


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
Invoking **ad-explore** for Task Tracker:

- **What exists**  
  - `src/models/task.ts`: `Task` with `id`, `title`, `status`, timestamps; `TaskFilter` supports `status` + `search`.  
  - `src/services/task-service.ts`: in-memory array, `addTask`, `listTasks(filter)`, `getFormattedTasks`. Filtering only by status/search; no sorting.  
  - `src/utils/format-task.ts`: shared formatter for CLI/API/reports plus `derivePriority` helper (returns `'high' | 'medium' | 'low'` but unused).  
  - Entry points: CLI (`src/cli/index.ts`) exposes `add` & `list`; API (`src/api/index.ts`) exposes GET/POST `/tasks` mirroring service; `src/reports/summary.ts` also consumes `formatTask`.

- **What needs to change**  
  - Extend `Task` model (and likely `TaskFilter`) to capture explicit priority.  
  - Update service layer to store priority, allow sorting by priority (likely with new filter arg).  
  - Update CLI/API commands to accept priority on create (and optionally filtering) plus ability to request priority-sorted output.  
  - Ensure `formatTask` (and summary/report consumers) display or otherwise reflect priority.  
  - Align `derivePriority` or remove/rework.

- **Consumers of shared interfaces**  
  - `Task` used by services, CLI, API, utils, reports.  
  - `formatTask` called from CLI (`getFormattedTasks`), API (`map` before returning body), reports (`generateSummary`). Changes ripple through all.

- **Key decisions pending**  
  - Priority enum (e.g., high/medium/low) and defaults.  
  - How users specify priority in CLI command and API payload.  
  - Sorting semantics (always sort by priority? optional flag? tie-breaker).  
  - Backward compatibility for tasks without explicit priority (default?).  
  - Whether to unify or remove `derivePriority`.

- **Risks**  
  - Multi-consumer impact when changing formatter output or Task shape.  
  - Ensuring CLI/API stay in sync; potential breaking changes for existing automation/tests.  
  - Sorting determinism when priorities equal (need secondary sort).