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
Design a priority field for tasks. Approach B sounds good, let's go with that.

[[Assistant]]:
I'll write up the design spec based on Approach B.

[[User]]:
Yes, write the spec.

@[assistant]:


@[toolResult]:
/home/christso/.agentv/workspaces/5dac7149-3739-42c7-83bf-d6036399f8ee/shared


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
api
cli
models
reports
services
utils


@[assistant]:


@[toolResult]:
shared


@[assistant]:


@[toolResult]:
ls: cannot access '../task-tracker.worktrees': No such file or directory


Command exited with code 2

@[assistant]:


@[toolResult]:
feat
feat-agentic-workflows
fix


@[assistant]:


@[toolResult]:
AGENTS.md
apps
biome.json
bun.lock
CLAUDE.md
CONTRIBUTING.md
docker-compose.yml
Dockerfile
docs
evals
examples
LICENSE
node_modules
package.json
packages
plugins
README.md
scripts
tsconfig.base.json
tsconfig.build.json
tsconfig.eslint.json
tsconfig.json


@[assistant]:


@[toolResult]:
agentic-engineering
agentv-claude-trace
agentv-dev
autopilot-dev


@[assistant]:


@[toolResult]:
skills


@[assistant]:


@[toolResult]:
ap-claim
ap-design
ap-explore
ap-implement
ap-plan
ap-ship
ap-using-autopilot-dev
ap-verify


@[assistant]:


@[toolResult]:
ls: cannot access 'plugins': No such file or directory


Command exited with code 2

@[assistant]:


@[toolResult]:
## feat/42-add-priority
AM prompt.md


@[assistant]:


@[toolResult]:
@[User]:
<file: path="plugins/autopilot-dev/skills/ap-design/SKILL.md">

@[User]:
Design a priority field for tasks. Approach B sounds good, let's go with that.

@[Assistant]:
I'll write up the design spec based on Approach B.

@[User]:
Yes, write the spec.

@[assistant]:


@[toolResult]:
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


@[assistant]:


@[toolResult]:
.
..
.agents
AGENTS.md
biome.json
.claude
CLAUDE.md
.git
package.json
.pi
prompt.md
README.md
scripts
src
tsconfig.json


@[assistant]:


@[toolResult]:
skills


@[assistant]:


@[toolResult]:
.agents:
skills

.agents/skills:
ap-claim
ap-design
ap-explore
ap-implement
ap-plan
ap-ship
ap-using-autopilot-dev
ap-verify

.agents/skills/ap-claim:
SKILL.md

.agents/skills/ap-design:
SKILL.md

.agents/skills/ap-explore:
SKILL.md

.agents/skills/ap-implement:
references
SKILL.md

.agents/skills/ap-implement/references:
code-quality-reviewer-prompt.md
implementer-prompt.md
spec-reviewer-prompt.md

.agents/skills/ap-plan:
SKILL.md

.agents/skills/ap-ship:
SKILL.md

.agents/skills/ap-using-autopilot-dev:
SKILL.md

.agents/skills/ap-verify:
SKILL.md


@[assistant]:


@[toolResult]:
ENOENT: no such file or directory, access '/home/christso/.agentv/workspaces/5dac7149-3739-42c7-83bf-d6036399f8ee/shared/src/models/index.ts'

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
skills


@[assistant]:


@[toolResult]:
skills


@[assistant]:


@[toolResult]:
ap-claim
ap-design
ap-explore
ap-implement
ap-plan
ap-ship
ap-using-autopilot-dev
ap-verify


@[assistant]:


@[toolResult]:
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


@[assistant]:


@[toolResult]:
ls: cannot access '.agents/plans': No such file or directory


Command exited with code 2

@[assistant]:


@[toolResult]:
skills


@[assistant]:


@[toolResult]:
(no output)

@[assistant]:


@[toolResult]:
(no output)

@[assistant]:


@[toolResult]:
2026-03-29


@[assistant]:


@[toolResult]:
Successfully wrote 6696 bytes to .agents/plans/2026-03-29-priority-field-design.md

@[assistant]:
Design spec written to `.agents/plans/2026-03-29-priority-field-design.md`. Let me know if you’d like any revisions before we proceed.