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
I've finished implementing the priority field feature.
All the code is written. Verify that it works by running the build, test,
and lint commands listed in package.json.


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
/home/christso/.agentv/workspaces/7ce0a34d-47f1-401b-bca8-2532da3c2c35/shared


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


@[toolResult]:

> task-tracker@1.0.0 build
> tsc --noEmit



@[assistant]:


@[toolResult]:

> task-tracker@1.0.0 test
> vitest run

sh: 1: vitest: not found


Command exited with code 127

@[assistant]:


@[toolResult]:

added 53 packages, and audited 54 packages in 7s

16 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities


@[assistant]:


@[toolResult]:

> task-tracker@1.0.0 test
> vitest run


 RUN  v3.2.4 /home/christso/.agentv/workspaces/7ce0a34d-47f1-401b-bca8-2532da3c2c35/shared

No test files found, exiting with code 1

include: **/*.{test,spec}.?(c|m)[jt]s?(x)
exclude:  **/node_modules/**, **/dist/**, **/cypress/**, **/.{idea,git,cache,output,temp}/**, **/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build,eslint,prettier}.config.*



Command exited with code 1

@[assistant]:


@[toolResult]:

> task-tracker@1.0.0 lint
> biome check .

./src/cli/index.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     1    │ - import·{·addTask,·getFormattedTasks·}·from·'../services/task-service';
     2    │ - import·{·formatTask·}·from·'../utils/format-task';
        1 │ + import·{·addTask,·getFormattedTasks·}·from·"../services/task-service";
        2 │ + import·{·formatTask·}·from·"../utils/format-task";
     3  3 │   
     4  4 │   export function runCli(args: string[]): string {
    ····· │ 
     6  6 │   
     7  7 │     switch (command) {
     8    │ - ····case·'add':·{
     9    │ - ······const·task·=·addTask(rest.join('·'));
        8 │ + ····case·"add":·{
        9 │ + ······const·task·=·addTask(rest.join("·"));
    10 10 │         return `Created: ${formatTask(task)}`;
    11 11 │       }
    12    │ - ····case·'list':
    13    │ - ······return·getFormattedTasks().join('\n')·||·'No·tasks·found.';
       12 │ + ····case·"list":
       13 │ + ······return·getFormattedTasks().join("\n")·||·"No·tasks·found.";
    14 14 │       default:
    15    │ - ······return·'Usage:·task-tracker·[add|list]·[args...]';
       15 │ + ······return·"Usage:·task-tracker·[add|list]·[args...]";
    16 16 │     }
    17 17 │   }
  

./src/utils/format-task.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     1    │ - import·type·{·Task·}·from·'../models/task';
        1 │ + import·type·{·Task·}·from·"../models/task";
     2  2 │   
     3  3 │   /**
    ····· │ 
     6  6 │    */
     7  7 │   export function formatTask(task: Task): string {
     8    │ - ··const·statusIcon·=·task.status·===·'done'·?·'✓'·:·task.status·===·'in_progress'·?·'→'·:·'○';
        8 │ + ··const·statusIcon·=
        9 │ + ····task.status·===·"done"·?·"✓"·:·task.status·===·"in_progress"·?·"→"·:·"○";
     9 10 │     return `${statusIcon} [${task.id}] ${task.title}`;
    10 11 │   }
    ····· │ 
    14 15 │    * Currently only handles basic cases — does not support custom priority rules.
    15 16 │    */
    16    │ - export·function·derivePriority(task:·Task):·'high'·|·'medium'·|·'low'·{
    17    │ - ··if·(task.status·===·'in_progress')·return·'high';
    18    │ - ··return·'medium';
       17 │ + export·function·derivePriority(task:·Task):·"high"·|·"medium"·|·"low"·{
       18 │ + ··if·(task.status·===·"in_progress")·return·"high";
       19 │ + ··return·"medium";
    19 20 │   }
    20 21 │   
  

./src/api/index.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     1    │ - import·{·addTask,·listTasks·}·from·'../services/task-service';
     2    │ - import·{·formatTask·}·from·'../utils/format-task';
        1 │ + import·{·addTask,·listTasks·}·from·"../services/task-service";
        2 │ + import·{·formatTask·}·from·"../utils/format-task";
     3  3 │   
     4  4 │   /**
    ····· │ 
    11 11 │     body?: Record<string, unknown>,
    12 12 │   ): { status: number; body: unknown } {
    13    │ - ··if·(method·===·'GET'·&&·path·===·'/tasks')·{
       13 │ + ··if·(method·===·"GET"·&&·path·===·"/tasks")·{
    14 14 │       const tasks = listTasks();
    15 15 │       return { status: 200, body: tasks.map(formatTask) };
    16 16 │     }
    17 17 │   
    18    │ - ··if·(method·===·'POST'·&&·path·===·'/tasks')·{
       18 │ + ··if·(method·===·"POST"·&&·path·===·"/tasks")·{
    19 19 │       const title = body?.title as string;
    20    │ - ····if·(!title)·return·{·status:·400,·body:·{·error:·'title·is·required'·}·};
       20 │ + ····if·(!title)·return·{·status:·400,·body:·{·error:·"title·is·required"·}·};
    21 21 │       const task = addTask(title);
    22 22 │       return { status: 201, body: task };
    23 23 │     }
    24 24 │   
    25    │ - ··return·{·status:·404,·body:·{·error:·'not·found'·}·};
       25 │ + ··return·{·status:·404,·body:·{·error:·"not·found"·}·};
    26 26 │   }
    27 27 │   
  

./src/models/task.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     2  2 │     readonly id: string;
     3  3 │     readonly title: string;
     4    │ - ··readonly·status:·'todo'·|·'in_progress'·|·'done';
        4 │ + ··readonly·status:·"todo"·|·"in_progress"·|·"done";
     5  5 │     readonly createdAt: Date;
     6  6 │     readonly updatedAt: Date;
    ····· │ 
     8  8 │   
     9  9 │   export interface TaskFilter {
    10    │ - ··readonly·status?:·Task['status'];
       10 │ + ··readonly·status?:·Task["status"];
    11 11 │     readonly search?: string;
    12 12 │   }
  

./src/reports/summary.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     1    │ - import·{·listTasks·}·from·'../services/task-service';
     2    │ - import·{·formatTask·}·from·'../utils/format-task';
        1 │ + import·{·listTasks·}·from·"../services/task-service";
        2 │ + import·{·formatTask·}·from·"../utils/format-task";
     3  3 │   
     4  4 │   /**
    ····· │ 
     7  7 │   export function generateSummary(): string {
     8  8 │     const tasks = listTasks();
     9    │ - ··const·lines·=·[`Task·Summary·(${tasks.length}·total)`,·'---',·...tasks.map(formatTask)];
    10    │ - ··return·lines.join('\n');
        9 │ + ··const·lines·=·[
       10 │ + ····`Task·Summary·(${tasks.length}·total)`,
       11 │ + ····"---",
       12 │ + ····...tasks.map(formatTask),
       13 │ + ··];
       14 │ + ··return·lines.join("\n");
    11 15 │   }
    12 16 │   
  

./scripts/setup.mjs format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     5  5 │    */
     6  6 │   
     7    │ - import·{·execSync·}·from·'node:child_process';
     8    │ - import·{·cpSync,·mkdirSync,·readFileSync,·readdirSync·}·from·'node:fs';
     9    │ - import·{·join·}·from·'node:path';
        7 │ + import·{·execSync·}·from·"node:child_process";
        8 │ + import·{·cpSync,·mkdirSync,·readFileSync,·readdirSync·}·from·"node:fs";
        9 │ + import·{·join·}·from·"node:path";
    10 10 │   
    11 11 │   // Read workspace_path from stdin (provided by AgentV orchestrator)
    12 12 │   let workspacePath;
    13 13 │   try {
    14    │ - ··const·stdin·=·readFileSync(0,·'utf8');
       14 │ + ··const·stdin·=·readFileSync(0,·"utf8");
    15 15 │     const context = JSON.parse(stdin);
    16 16 │     workspacePath = context.workspace_path;
    ····· │ 
    22 22 │   let repoRoot;
    23 23 │   try {
    24    │ - ··repoRoot·=·execSync('git·rev-parse·--show-toplevel',·{
    25    │ - ····encoding:·'utf8',
       24 │ + ··repoRoot·=·execSync("git·rev-parse·--show-toplevel",·{
       25 │ + ····encoding:·"utf8",
    26 26 │     }).trim();
    27 27 │   } catch {
    28    │ - ··console.error('Failed·to·resolve·repo·root·from·cwd:',·process.cwd());
       28 │ + ··console.error("Failed·to·resolve·repo·root·from·cwd:",·process.cwd());
    29 29 │     process.exit(1);
    30 30 │   }
    ····· │ 
    39 39 │   //   Codex: .agents/skills/ or .codex/skills/
    40 40 │   const skillDirs = [
    41    │ - ··join(workspacePath,·'.claude',·'skills'),
    42    │ - ··join(workspacePath,·'.agents',·'skills'),
    43    │ - ··join(workspacePath,·'.pi',·'skills'),
       41 │ + ··join(workspacePath,·".claude",·"skills"),
       42 │ + ··join(workspacePath,·".agents",·"skills"),
       43 │ + ··join(workspacePath,·".pi",·"skills"),
    44 44 │   ];
    45 45 │   for (const dir of skillDirs) {
    ····· │ 
    48 48 │   
    49 49 │   // Copy all autopilot-dev skills
    50    │ - const·pluginSkillsDir·=·join(repoRoot,·'plugins',·'autopilot-dev',·'skills');
       50 │ + const·pluginSkillsDir·=·join(repoRoot,·"plugins",·"autopilot-dev",·"skills");
    51 51 │   const skillNames = readdirSync(pluginSkillsDir);
    52 52 │   
    ····· │ 
    60 60 │   
    61 61 │   for (const dir of skillDirs) {
    62    │ - ··console.log(`Skills·in·${dir}:·${readdirSync(dir).join(',·')}`);
       62 │ + ··console.log(`Skills·in·${dir}:·${readdirSync(dir).join(",·")}`);
    63 63 │   }
    64 64 │   
    ····· │ 
    67 67 │     execSync('git init && git add -A && git commit -m "initial commit"', {
    68 68 │       cwd: workspacePath,
    69    │ - ····encoding:·'utf8',
    70    │ - ····stdio:·'pipe',
       69 │ + ····encoding:·"utf8",
       70 │ + ····stdio:·"pipe",
    71 71 │     });
    72    │ - ··execSync('git·checkout·-b·feat/42-add-priority',·{
       72 │ + ··execSync("git·checkout·-b·feat/42-add-priority",·{
    73 73 │       cwd: workspacePath,
    74    │ - ····encoding:·'utf8',
    75    │ - ····stdio:·'pipe',
       74 │ + ····encoding:·"utf8",
       75 │ + ····stdio:·"pipe",
    76 76 │     });
    77    │ - ··console.log('Git·repo·initialized·with·feat·branch');
       77 │ + ··console.log("Git·repo·initialized·with·feat·branch");
    78 78 │   } catch (e) {
    79    │ - ··console.error('Git·init·failed:',·e.message);
       79 │ + ··console.error("Git·init·failed:",·e.message);
    80 80 │   }
    81 81 │   
  

./src/services/task-service.ts format ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Formatter would have printed the following content:
  
     1    │ - import·type·{·Task,·TaskFilter·}·from·'../models/task';
     2    │ - import·{·formatTask·}·from·'../utils/format-task';
        1 │ + import·type·{·Task,·TaskFilter·}·from·"../models/task";
        2 │ + import·{·formatTask·}·from·"../utils/format-task";
     3  3 │   
     4  4 │   const tasks: Task[] = [];
    ····· │ 
     8  8 │       id: `task-${tasks.length + 1}`,
     9  9 │       title,
    10    │ - ····status:·'todo',
       10 │ + ····status:·"todo",
    11 11 │       createdAt: new Date(),
    12 12 │       updatedAt: new Date(),
  

check ━━━━━Checked 10 files in 10ms. No fixes applied.
Found 7 errors.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Some errors were emitted while running checks.
  



Command exited with code 1

@[assistant]:
Verification results (requested `npm run build`, `npm run test`, `npm run lint`):

- `npm run build` (tsc --noEmit): ✅ Passed with no errors.
- `npm run test` (vitest run): ❌ Failed because Vitest found no test files (`include: **/*.{test,spec}...`). Exit code 1.
- `npm run lint` (biome check .): ❌ Failed — Biome reported formatting differences in multiple files (e.g., `src/cli/index.ts`, `src/utils/format-task.ts`, `src/api/index.ts`, `src/models/task.ts`, `src/reports/summary.ts`, `scripts/setup.mjs`, `src/services/task-service.ts`). It wants double quotes and reformatted multiline constructs.

Please add at least one Vitest spec (or adjust the config) so the test command succeeds, and run `biome format`/apply the suggested formatting so the lint step is clean. Once those are resolved, re-run the three commands to obtain passing evidence.