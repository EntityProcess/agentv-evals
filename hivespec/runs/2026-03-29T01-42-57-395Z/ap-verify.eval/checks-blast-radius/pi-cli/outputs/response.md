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
I modified the Task interface in src/models/task.ts and the formatTask function
in src/utils/format-task.ts. Check whether any consumers were missed.


@[assistant]:


@[toolResult]:
/home/christso/.agentv/workspaces/7ce0a34d-47f1-401b-bca8-2532da3c2c35/shared


@[assistant]:


@[toolResult]:
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


@[assistant]:


@[toolResult]:
AGENTS.md
biome.json
CLAUDE.md
node_modules
package.json
package-lock.json
prompt.md
README.md
scripts
src
tsconfig.json


@[assistant]:


@[toolResult]:
src/utils/format-task.ts:1:import type { Task } from '../models/task';
src/utils/format-task.ts:7:export function formatTask(task: Task): string {
src/utils/format-task.ts:16:export function derivePriority(task: Task): 'high' | 'medium' | 'low' {
src/services/task-service.ts:1:import type { Task, TaskFilter } from '../models/task';
src/services/task-service.ts:4:const tasks: Task[] = [];
src/services/task-service.ts:6:export function addTask(title: string): Task {
src/services/task-service.ts:7:  const task: Task = {
src/services/task-service.ts:18:export function listTasks(filter?: TaskFilter): Task[] {
src/models/task.ts:1:export interface Task {
src/models/task.ts:10:  readonly status?: Task['status'];
src/reports/summary.ts:9:  const lines = [`Task Summary (${tasks.length} total)`, '---', ...tasks.map(formatTask)];


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
## feat/42-add-priority
A  node_modules/.bin/biome
A  node_modules/.bin/esbuild
A  node_modules/.bin/nanoid
A  node_modules/.bin/rollup
A  node_modules/.bin/tsc
A  node_modules/.bin/tsserver
A  node_modules/.bin/vite
A  node_modules/.bin/vite-node
A  node_modules/.bin/vitest
A  node_modules/.bin/why-is-node-running
A  node_modules/.package-lock.json
A  node_modules/@biomejs/biome/LICENSE-APACHE
A  node_modules/@biomejs/biome/LICENSE-MIT
A  node_modules/@biomejs/biome/README.hi.md
A  node_modules/@biomejs/biome/README.ja.md
A  node_modules/@biomejs/biome/README.kr.md
A  node_modules/@biomejs/biome/README.md
A  node_modules/@biomejs/biome/README.pt-BR.md
A  node_modules/@biomejs/biome/README.zh-CN.md
A  node_modules/@biomejs/biome/README.zh-TW.md
A  node_modules/@biomejs/biome/ROME-LICENSE-MIT
A  node_modules/@biomejs/biome/bin/biome
A  node_modules/@biomejs/biome/configuration_schema.json
A  node_modules/@biomejs/biome/package.json
A  node_modules/@biomejs/biome/scripts/postinstall.js
A  node_modules/@biomejs/cli-linux-arm64/biome
A  node_modules/@biomejs/cli-linux-arm64/package.json
A  node_modules/@esbuild/linux-arm64/README.md
A  node_modules/@esbuild/linux-arm64/bin/esbuild
A  node_modules/@esbuild/linux-arm64/package.json
A  node_modules/@jridgewell/sourcemap-codec/LICENSE
A  node_modules/@jridgewell/sourcemap-codec/README.md
A  node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.mjs
A  node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.mjs.map
A  node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.umd.js
A  node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.umd.js.map
A  node_modules/@jridgewell/sourcemap-codec/package.json
A  node_modules/@jridgewell/sourcemap-codec/src/scopes.ts
A  node_modules/@jridgewell/sourcemap-codec/src/sourcemap-codec.ts
A  node_modules/@jridgewell/sourcemap-codec/src/strings.ts
A  node_modules/@jridgewell/sourcemap-codec/src/vlq.ts
A  node_modules/@jridgewell/sourcemap-codec/types/scopes.d.cts
A  node_modules/@jridgewell/sourcemap-codec/types/scopes.d.cts.map
A  node_modules/@jridgewell/sourcemap-codec/types/scopes.d.mts
A  node_modules/@jridgewell/sourcemap-codec/types/scopes.d.mts.map
A  node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.cts
A  node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.cts.map
A  node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.mts
A  node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.mts.map
A  node_modules/@jridgewell/sourcemap-codec/types/strings.d.cts
A  node_modules/@jridgewell/sourcemap-codec/types/strings.d.cts.map
A  node_modules/@jridgewell/sourcemap-codec/types/strings.d.mts
A  node_modules/@jridgewell/sourcemap-codec/types/strings.d.mts.map
A  node_modules/@jridgewell/sourcemap-codec/types/vlq.d.cts
A  node_modules/@jridgewell/sourcemap-codec/types/vlq.d.cts.map
A  node_modules/@jridgewell/sourcemap-codec/types/vlq.d.mts
A  node_modules/@jridgewell/sourcemap-codec/types/vlq.d.mts.map
A  node_modules/@rollup/rollup-linux-arm64-gnu/README.md
A  node_modules/@rollup/rollup-linux-arm64-gnu/package.json
A  node_modules/@rollup/rollup-linux-arm64-gnu/rollup.linux-arm64-gnu.node
A  node_modules/@types/chai/LICENSE
A  node_modules/@types/chai/README.md
A  node_modules/@types/chai/index.d.ts
A  node_modules/@types/chai/package.json
A  node_modules/@types/chai/register-should.d.ts
A  node_modules/@types/deep-eql/LICENSE
A  node_modules/@types/deep-eql/README.md
A  node_modules/@types/deep-eql/index.d.ts
A  node_modules/@types/deep-eql/package.json
A  node_modules/@types/estree/LICENSE
A  node_modules/@types/estree/README.md
A  node_modules/@types/estree/flow.d.ts
A  node_modules/@types/estree/index.d.ts
A  node_modules/@types/estree/package.json
A  node_modules/@vitest/expect/LICENSE
A  node_modules/@vitest/expect/README.md
A  node_modules/@vitest/expect/dist/index.d.ts
A  node_modules/@vitest/expect/dist/index.js
A  node_modules/@vitest/expect/package.json
A  node_modules/@vitest/mocker/LICENSE
A  node_modules/@vitest/mocker/README.md
A  node_modules/@vitest/mocker/dist/auto-register.d.ts
A  node_modules/@vitest/mocker/dist/auto-register.js
A  node_modules/@vitest/mocker/dist/browser.d.ts
A  node_modules/@vitest/mocker/dist/browser.js
A  node_modules/@vitest/mocker/dist/chunk-interceptor-native.js
A  node_modules/@vitest/mocker/dist/chunk-mocker.js
A  node_modules/@vitest/mocker/dist/chunk-pathe.M-eThtNZ.js
A  node_modules/@vitest/mocker/dist/chunk-registry.js
A  node_modules/@vitest/mocker/dist/chunk-utils.js
A  node_modules/@vitest/mocker/dist/index.d.ts
A  node_modules/@vitest/mocker/dist/index.js
A  node_modules/@vitest/mocker/dist/mocker.d-Ce9_ySj5.d.ts
A  node_modules/@vitest/mocker/dist/node.d.ts
A  node_modules/@vitest/mocker/dist/node.js
A  node_modules/@vitest/mocker/dist/redirect.d.ts
A  node_modules/@vitest/mocker/dist/redirect.js
A  node_modules/@vitest/mocker/dist/register.d.ts
A  node_modules/@vitest/mocker/dist/register.js
A  node_modules/@vitest/mocker/dist/registry.d-D765pazg.d.ts
A  node_modules/@vitest/mocker/dist/types.d-D_aRZRdy.d.ts
A  node_modules/@vitest/mocker/package.json
A  node_modules/@vitest/pretty-format/LICENSE
A  node_modules/@vitest/pretty-format/dist/index.d.ts
A  node_modules/@vitest/pretty-format/dist/index.js
A  node_modules/@vitest/pretty-format/package.json
A  node_modules/@vitest/runner/LICENSE
A  node_modules/@vitest/runner/README.md
A  node_modules/@vitest/runner/dist/chunk-hooks.js
A  node_modules/@vitest/runner/dist/index.d.ts
A  node_modules/@vitest/runner/dist/index.js
A  node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts
A  node_modules/@vitest/runner/dist/types.d.ts
A  node_modules/@vitest/runner/dist/types.js
A  node_modules/@vitest/runner/dist/utils.d.ts
A  node_modules/@vitest/runner/dist/utils.js
A  node_modules/@vitest/runner/package.json
A  node_modules/@vitest/runner/types.d.ts
A  node_modules/@vitest/runner/utils.d.ts
A  node_modules/@vitest/snapshot/LICENSE
A  node_modules/@vitest/snapshot/README.md
A  node_modules/@vitest/snapshot/dist/environment.d-DHdQ1Csl.d.ts
A  node_modules/@vitest/snapshot/dist/environment.d.ts
A  node_modules/@vitest/snapshot/dist/environment.js
A  node_modules/@vitest/snapshot/dist/index.d.ts
A  node_modules/@vitest/snapshot/dist/index.js
A  node_modules/@vitest/snapshot/dist/manager.d.ts
A  node_modules/@vitest/snapshot/dist/manager.js
A  node_modules/@vitest/snapshot/dist/rawSnapshot.d-lFsMJFUd.d.ts
A  node_modules/@vitest/snapshot/environment.d.ts
A  node_modules/@vitest/snapshot/manager.d.ts
A  node_modules/@vitest/snapshot/package.json
A  node_modules/@vitest/spy/LICENSE
A  node_modules/@vitest/spy/README.md
A  node_modules/@vitest/spy/dist/index.d.ts
A  node_modules/@vitest/spy/dist/index.js
A  node_modules/@vitest/spy/package.json
A  node_modules/@vitest/utils/LICENSE
A  node_modules/@vitest/utils/diff.d.ts
A  node_modules/@vitest/utils/dist/chunk-_commonjsHelpers.js
A  node_modules/@vitest/utils/dist/diff.d.ts
A  node_modules/@vitest/utils/dist/diff.js
A  node_modules/@vitest/utils/dist/error.d.ts
A  node_modules/@vitest/utils/dist/error.js
A  node_modules/@vitest/utils/dist/helpers.d.ts
A  node_modules/@vitest/utils/dist/helpers.js
A  node_modules/@vitest/utils/dist/index.d.ts
A  node_modules/@vitest/utils/dist/index.js
A  node_modules/@vitest/utils/dist/source-map.d.ts
A  node_modules/@vitest/utils/dist/source-map.js
A  node_modules/@vitest/utils/dist/types.d-BCElaP-c.d.ts
A  node_modules/@vitest/utils/dist/types.d.ts
A  node_modules/@vitest/utils/dist/types.js
A  node_modules/@vitest/utils/error.d.ts
A  node_modules/@vitest/utils/helpers.d.ts
A  node_modules/@vitest/utils/package.json
A  node_modules/assertion-error/LICENSE
A  node_modules/assertion-error/README.md
A  node_modules/assertion-error/index.d.ts
A  node_modules/assertion-error/index.js
A  node_modules/assertion-error/package.json
A  node_modules/cac/LICENSE
A  node_modules/cac/README.md
A  node_modules/cac/deno/CAC.ts
A  node_modules/cac/deno/Command.ts
A  node_modules/cac/deno/Option.ts
A  node_modules/cac/deno/deno.ts
A  node_modules/cac/deno/index.ts
A  node_modules/cac/deno/utils.ts
A  node_modules/cac/dist/index.d.ts
A  node_modules/cac/dist/index.js
A  node_modules/cac/dist/index.mjs
A  node_modules/cac/index-compat.js
A  node_modules/cac/mod.js
A  node_modules/cac/mod.ts
A  node_modules/cac/package.json
A  node_modules/chai/LICENSE
A  node_modules/chai/README.md
A  node_modules/chai/chai.js
A  node_modules/chai/index.js
A  node_modules/chai/lib/chai.js
A  node_modules/chai/lib/chai/assertion.js
A  node_modules/chai/lib/chai/config.js
A  node_modules/chai/lib/chai/core/assertions.js
A  node_modules/chai/lib/chai/interface/assert.js
A  node_modules/chai/lib/chai/interface/expect.js
A  node_modules/chai/lib/chai/interface/should.js
A  node_modules/chai/lib/chai/utils/addChainableMethod.js
A  node_modules/chai/lib/chai/utils/addLengthGuard.js
A  node_modules/chai/lib/chai/utils/addMethod.js
A  node_modules/chai/lib/chai/utils/addProperty.js
A  node_modules/chai/lib/chai/utils/compareByInspect.js
A  node_modules/chai/lib/chai/utils/expectTypes.js
A  node_modules/chai/lib/chai/utils/flag.js
A  node_modules/chai/lib/chai/utils/getActual.js
A  node_modules/chai/lib/chai/utils/getMessage.js
A  node_modules/chai/lib/chai/utils/getOperator.js
A  node_modules/chai/lib/chai/utils/getOwnEnumerableProperties.js
A  node_modules/chai/lib/chai/utils/getOwnEnumerablePropertySymbols.js
A  node_modules/chai/lib/chai/utils/getProperties.js
A  node_modules/chai/lib/chai/utils/index.js
A  node_modules/chai/lib/chai/utils/inspect.js
A  node_modules/chai/lib/chai/utils/isNaN.js
A  node_modules/chai/lib/chai/utils/isProxyEnabled.js
A  node_modules/chai/lib/chai/utils/objDisplay.js
A  node_modules/chai/lib/chai/utils/overwriteChainableMethod.js
A  node_modules/chai/lib/chai/utils/overwriteMethod.js
A  node_modules/chai/lib/chai/utils/overwriteProperty.js
A  node_modules/chai/lib/chai/utils/proxify.js
A  node_modules/chai/lib/chai/utils/test.js
A  node_modules/chai/lib/chai/utils/transferFlags.js
A  node_modules/chai/lib/chai/utils/type-detect.js
A  node_modules/chai/package.json
A  node_modules/chai/register-assert.js
A  node_modules/chai/register-expect.js
A  node_modules/chai/register-should.js
A  node_modules/check-error/LICENSE
A  node_modules/check-error/README.md
A  node_modules/check-error/index.js
A  node_modules/check-error/package.json
A  node_modules/debug/LICENSE
A  node_modules/debug/README.md
A  node_modules/debug/package.json
A  node_modules/debug/src/browser.js
A  node_modules/debug/src/common.js
A  node_modules/debug/src/index.js
A  node_modules/debug/src/node.js
A  node_modules/deep-eql/LICENSE
A  node_modules/deep-eql/README.md
A  node_modules/deep-eql/index.js
A  node_modules/deep-eql/package.json
A  node_modules/es-module-lexer/LICENSE
A  node_modules/es-module-lexer/README.md
A  node_modules/es-module-lexer/dist/lexer.asm.js
A  node_modules/es-module-lexer/dist/lexer.cjs
A  node_modules/es-module-lexer/dist/lexer.js
A  node_modules/es-module-lexer/lexer.js
A  node_modules/es-module-lexer/package.json
A  node_modules/es-module-lexer/types/lexer.d.ts
A  node_modules/esbuild/LICENSE.md
A  node_modules/esbuild/README.md
A  node_modules/esbuild/bin/esbuild
A  node_modules/esbuild/install.js
A  node_modules/esbuild/lib/main.d.ts
A  node_modules/esbuild/lib/main.js
A  node_modules/esbuild/package.json
A  node_modules/estree-walker/LICENSE
A  node_modules/estree-walker/README.md
A  node_modules/estree-walker/package.json
A  node_modules/estree-walker/src/async.js
A  node_modules/estree-walker/src/index.js
A  node_modules/estree-walker/src/sync.js
A  node_modules/estree-walker/src/walker.js
A  node_modules/estree-walker/types/async.d.ts
A  node_modules/estree-walker/types/index.d.ts
A  node_modules/estree-walker/types/sync.d.ts
A  node_modules/estree-walker/types/walker.d.ts
A  node_modules/expect-type/LICENSE
A  node_modules/expect-type/README.md
A  node_modules/expect-type/SECURITY.md
A  node_modules/expect-type/dist/branding.d.ts
A  node_modules/expect-type/dist/branding.js
A  node_modules/expect-type/dist/index.d.ts
A  node_modules/expect-type/dist/index.js
A  node_modules/expect-type/dist/messages.d.ts
A  node_modules/expect-type/dist/messages.js
A  node_modules/expect-type/dist/overloads.d.ts
A  node_modules/expect-type/dist/overloads.js
A  node_modules/expect-type/dist/utils.d.ts
A  node_modules/expect-type/dist/utils.js
A  node_modules/expect-type/package.json
A  node_modules/fdir/LICENSE
A  node_modules/fdir/README.md
A  node_modules/fdir/dist/index.cjs
A  node_modules/fdir/dist/index.d.cts
A  node_modules/fdir/dist/index.d.mts
A  node_modules/fdir/dist/index.mjs
A  node_modules/fdir/package.json
A  node_modules/js-tokens/LICENSE
A  node_modules/js-tokens/README.md
A  node_modules/js-tokens/index.d.ts
A  node_modules/js-tokens/index.js
A  node_modules/js-tokens/package.json
A  node_modules/loupe/LICENSE
A  node_modules/loupe/README.md
A  node_modules/loupe/lib/arguments.d.ts
A  node_modules/loupe/lib/arguments.d.ts.map
A  node_modules/loupe/lib/arguments.js
A  node_modules/loupe/lib/array.d.ts
A  node_modules/loupe/lib/array.d.ts.map
A  node_modules/loupe/lib/array.js
A  node_modules/loupe/lib/bigint.d.ts
A  node_modules/loupe/lib/bigint.d.ts.map
A  node_modules/loupe/lib/bigint.js
A  node_modules/loupe/lib/class.d.ts
A  node_modules/loupe/lib/class.d.ts.map
A  node_modules/loupe/lib/class.js
A  node_modules/loupe/lib/date.d.ts
A  node_modules/loupe/lib/date.d.ts.map
A  node_modules/loupe/lib/date.js
A  node_modules/loupe/lib/error.d.ts
A  node_modules/loupe/lib/error.d.ts.map
A  node_modules/loupe/lib/error.js
A  node_modules/loupe/lib/function.d.ts
A  node_modules/loupe/lib/function.d.ts.map
A  node_modules/loupe/lib/function.js
A  node_modules/loupe/lib/helpers.d.ts
A  node_modules/loupe/lib/helpers.d.ts.map
A  node_modules/loupe/lib/helpers.js
A  node_modules/loupe/lib/html.d.ts
A  node_modules/loupe/lib/html.d.ts.map
A  node_modules/loupe/lib/html.js
A  node_modules/loupe/lib/index.d.ts
A  node_modules/loupe/lib/index.d.ts.map
A  node_modules/loupe/lib/index.js
A  node_modules/loupe/lib/map.d.ts
A  node_modules/loupe/lib/map.d.ts.map
A  node_modules/loupe/lib/map.js
A  node_modules/loupe/lib/number.d.ts
A  node_modules/loupe/lib/number.d.ts.map
A  node_modules/loupe/lib/number.js
A  node_modules/loupe/lib/object.d.ts
A  node_modules/loupe/lib/object.d.ts.map
A  node_modules/loupe/lib/object.js
A  node_modules/loupe/lib/promise.d.ts
A  node_modules/loupe/lib/promise.d.ts.map
A  node_modules/loupe/lib/promise.js
A  node_modules/loupe/lib/regexp.d.ts
A  node_modules/loupe/lib/regexp.d.ts.map
A  node_modules/loupe/lib/regexp.js
A  node_modules/loupe/lib/set.d.ts
A  node_modules/loupe/lib/set.d.ts.map
A  node_modules/loupe/lib/set.js
A  node_modules/loupe/lib/string.d.ts
A  node_modules/loupe/lib/string.d.ts.map
A  node_modules/loupe/lib/string.js
A  node_modules/loupe/lib/symbol.d.ts
A  node_modules/loupe/lib/symbol.d.ts.map
A  node_modules/loupe/lib/symbol.js
A  node_modules/loupe/lib/typedarray.d.ts
A  node_modules/loupe/lib/typedarray.d.ts.map
A  node_modules/loupe/lib/typedarray.js
A  node_modules/loupe/lib/types.d.ts
A  node_modules/loupe/lib/types.d.ts.map
A  node_modules/loupe/lib/types.js
A  node_modules/loupe/loupe.js
A  node_modules/loupe/package.json
A  node_modules/magic-string/LICENSE
A  node_modules/magic-string/README.md
A  node_modules/magic-string/dist/magic-string.cjs.d.ts
A  node_modules/magic-string/dist/magic-string.cjs.js
A  node_modules/magic-string/dist/magic-string.cjs.js.map
A  node_modules/magic-string/dist/magic-string.es.d.mts
A  node_modules/magic-string/dist/magic-string.es.mjs
A  node_modules/magic-string/dist/magic-string.es.mjs.map
A  node_modules/magic-string/dist/magic-string.umd.js
A  node_modules/magic-string/dist/magic-string.umd.js.map
A  node_modules/magic-string/package.json
A  node_modules/ms/index.js
A  node_modules/ms/license.md
A  node_modules/ms/package.json
A  node_modules/ms/readme.md
A  node_modules/nanoid/LICENSE
A  node_modules/nanoid/README.md
A  node_modules/nanoid/async/index.browser.cjs
A  node_modules/nanoid/async/index.browser.js
A  node_modules/nanoid/async/index.cjs
A  node_modules/nanoid/async/index.d.ts
A  node_modules/nanoid/async/index.js
A  node_modules/nanoid/async/index.native.js
A  node_modules/nanoid/async/package.json
A  node_modules/nanoid/bin/nanoid.cjs
A  node_modules/nanoid/index.browser.cjs
A  node_modules/nanoid/index.browser.js
A  node_modules/nanoid/index.cjs
A  node_modules/nanoid/index.d.cts
A  node_modules/nanoid/index.d.ts
A  node_modules/nanoid/index.js
A  node_modules/nanoid/nanoid.js
A  node_modules/nanoid/non-secure/index.cjs
A  node_modules/nanoid/non-secure/index.d.ts
A  node_modules/nanoid/non-secure/index.js
A  node_modules/nanoid/non-secure/package.json
A  node_modules/nanoid/package.json
A  node_modules/nanoid/url-alphabet/index.cjs
A  node_modules/nanoid/url-alphabet/index.js
A  node_modules/nanoid/url-alphabet/package.json
A  node_modules/pathe/LICENSE
A  node_modules/pathe/README.md
A  node_modules/pathe/dist/index.cjs
A  node_modules/pathe/dist/index.d.cts
A  node_modules/pathe/dist/index.d.mts
A  node_modules/pathe/dist/index.d.ts
A  node_modules/pathe/dist/index.mjs
A  node_modules/pathe/dist/shared/pathe.BSlhyZSM.cjs
A  node_modules/pathe/dist/shared/pathe.M-eThtNZ.mjs
A  node_modules/pathe/dist/utils.cjs
A  node_modules/pathe/dist/utils.d.cts
A  node_modules/pathe/dist/utils.d.mts
A  node_modules/pathe/dist/utils.d.ts
A  node_modules/pathe/dist/utils.mjs
A  node_modules/pathe/package.json
A  node_modules/pathe/utils.d.ts
A  node_modules/pathval/LICENSE
A  node_modules/pathval/README.md
A  node_modules/pathval/index.js
A  node_modules/pathval/package.json
A  node_modules/picocolors/LICENSE
A  node_modules/picocolors/README.md
A  node_modules/picocolors/package.json
A  node_modules/picocolors/picocolors.browser.js
A  node_modules/picocolors/picocolors.d.ts
A  node_modules/picocolors/picocolors.js
A  node_modules/picocolors/types.d.ts
A  node_modules/picomatch/LICENSE
A  node_modules/picomatch/README.md
A  node_modules/picomatch/index.js
A  node_modules/picomatch/lib/constants.js
A  node_modules/picomatch/lib/parse.js
A  node_modules/picomatch/lib/picomatch.js
A  node_modules/picomatch/lib/scan.js
A  node_modules/picomatch/lib/utils.js
A  node_modules/picomatch/package.json
A  node_modules/picomatch/posix.js
A  node_modules/postcss/LICENSE
A  node_modules/postcss/README.md
A  node_modules/postcss/lib/at-rule.d.ts
A  node_modules/postcss/lib/at-rule.js
A  node_modules/postcss/lib/comment.d.ts
A  node_modules/postcss/lib/comment.js
A  node_modules/postcss/lib/container.d.ts
A  node_modules/postcss/lib/container.js
A  node_modules/postcss/lib/css-syntax-error.d.ts
A  node_modules/postcss/lib/css-syntax-error.js
A  node_modules/postcss/lib/declaration.d.ts
A  node_modules/postcss/lib/declaration.js
A  node_modules/postcss/lib/document.d.ts
A  node_modules/postcss/lib/document.js
A  node_modules/postcss/lib/fromJSON.d.ts
A  node_modules/postcss/lib/fromJSON.js
A  node_modules/postcss/lib/input.d.ts
A  node_modules/postcss/lib/input.js
A  node_modules/postcss/lib/lazy-result.d.ts
A  node_modules/postcss/lib/lazy-result.js
A  node_modules/postcss/lib/list.d.ts
A  node_modules/postcss/lib/list.js
A  node_modules/postcss/lib/map-generator.js
A  node_modules/postcss/lib/no-work-result.d.ts
A  node_modules/postcss/lib/no-work-result.js
A  node_modules/postcss/lib/node.d.ts
A  node_modules/postcss/lib/node.js
A  node_modules/postcss/lib/parse.d.ts
A  node_modules/postcss/lib/parse.js
A  node_modules/postcss/lib/parser.js
A  node_modules/postcss/lib/postcss.d.mts
A  node_modules/postcss/lib/postcss.d.ts
A  node_modules/postcss/lib/postcss.js
A  node_modules/postcss/lib/postcss.mjs
A  node_modules/postcss/lib/previous-map.d.ts
A  node_modules/postcss/lib/previous-map.js
A  node_modules/postcss/lib/processor.d.ts
A  node_modules/postcss/lib/processor.js
A  node_modules/postcss/lib/result.d.ts
A  node_modules/postcss/lib/result.js
A  node_modules/postcss/lib/root.d.ts
A  node_modules/postcss/lib/root.js
A  node_modules/postcss/lib/rule.d.ts
A  node_modules/postcss/lib/rule.js
A  node_modules/postcss/lib/stringifier.d.ts
A  node_modules/postcss/lib/stringifier.js
A  node_modules/postcss/lib/stringify.d.ts
A  node_modules/postcss/lib/stringify.js
A  node_modules/postcss/lib/symbols.js
A  node_modules/postcss/lib/terminal-highlight.js
A  node_modules/postcss/lib/tokenize.js
A  node_modules/postcss/lib/warn-once.js
A  node_modules/postcss/lib/warning.d.ts
A  node_modules/postcss/lib/warning.js
A  node_modules/postcss/package.json
A  node_modules/rollup/LICENSE.md
A  node_modules/rollup/README.md
A  node_modules/rollup/dist/bin/rollup
A  node_modules/rollup/dist/es/getLogFilter.js
A  node_modules/rollup/dist/es/package.json
A  node_modules/rollup/dist/es/parseAst.js
A  node_modules/rollup/dist/es/rollup.js
A  node_modules/rollup/dist/es/shared/node-entry.js
A  node_modules/rollup/dist/es/shared/parseAst.js
A  node_modules/rollup/dist/es/shared/watch.js
A  node_modules/rollup/dist/getLogFilter.d.ts
A  node_modules/rollup/dist/getLogFilter.js
A  node_modules/rollup/dist/loadConfigFile.d.ts
A  node_modules/rollup/dist/loadConfigFile.js
A  node_modules/rollup/dist/native.js
A  node_modules/rollup/dist/parseAst.d.ts
A  node_modules/rollup/dist/parseAst.js
A  node_modules/rollup/dist/rollup.d.ts
A  node_modules/rollup/dist/rollup.js
A  node_modules/rollup/dist/shared/fsevents-importer.js
A  node_modules/rollup/dist/shared/index.js
A  node_modules/rollup/dist/shared/loadConfigFile.js
A  node_modules/rollup/dist/shared/parseAst.js
A  node_modules/rollup/dist/shared/rollup.js
A  node_modules/rollup/dist/shared/watch-cli.js
A  node_modules/rollup/dist/shared/watch.js
A  node_modules/rollup/package.json
A  node_modules/siginfo/.travis.yml
A  node_modules/siginfo/LICENSE
A  node_modules/siginfo/README.md
A  node_modules/siginfo/index.js
A  node_modules/siginfo/package.json
A  node_modules/siginfo/test.js
A  node_modules/source-map-js/LICENSE
A  node_modules/source-map-js/README.md
A  node_modules/source-map-js/lib/array-set.js
A  node_modules/source-map-js/lib/base64-vlq.js
A  node_modules/source-map-js/lib/base64.js
A  node_modules/source-map-js/lib/binary-search.js
A  node_modules/source-map-js/lib/mapping-list.js
A  node_modules/source-map-js/lib/quick-sort.js
A  node_modules/source-map-js/lib/source-map-consumer.d.ts
A  node_modules/source-map-js/lib/source-map-consumer.js
A  node_modules/source-map-js/lib/source-map-generator.d.ts
A  node_modules/source-map-js/lib/source-map-generator.js
A  node_modules/source-map-js/lib/source-node.d.ts
A  node_modules/source-map-js/lib/source-node.js
A  node_modules/source-map-js/lib/util.js
A  node_modules/source-map-js/package.json
A  node_modules/source-map-js/source-map.d.ts
A  node_modules/source-map-js/source-map.js
A  node_modules/stackback/.npmignore
A  node_modules/stackback/.travis.yml
A  node_modules/stackback/README.md
A  node_modules/stackback/formatstack.js
A  node_modules/stackback/index.js
A  node_modules/stackback/package.json
A  node_modules/stackback/test.js
A  node_modules/std-env/LICENCE
A  node_modules/std-env/README.md
A  node_modules/std-env/dist/index.cjs
A  node_modules/std-env/dist/index.d.cts
A  node_modules/std-env/dist/index.d.mts
A  node_modules/std-env/dist/index.d.ts
A  node_modules/std-env/dist/index.mjs
A  node_modules/std-env/package.json
A  node_modules/strip-literal/LICENSE
A  node_modules/strip-literal/README.md
A  node_modules/strip-literal/dist/index.cjs
A  node_modules/strip-literal/dist/index.d.cts
A  node_modules/strip-literal/dist/index.d.mts
A  node_modules/strip-literal/dist/index.d.ts
A  node_modules/strip-literal/dist/index.mjs
A  node_modules/strip-literal/package.json
A  node_modules/tinybench/LICENSE
A  node_modules/tinybench/README.md
A  node_modules/tinybench/dist/index.cjs
A  node_modules/tinybench/dist/index.d.cts
A  node_modules/tinybench/dist/index.d.ts
A  node_modules/tinybench/dist/index.js
A  node_modules/tinybench/package.json
A  node_modules/tinyexec/LICENSE
A  node_modules/tinyexec/README.md
A  node_modules/tinyexec/dist/main.cjs
A  node_modules/tinyexec/dist/main.d.cts
A  node_modules/tinyexec/dist/main.d.ts
A  node_modules/tinyexec/dist/main.js
A  node_modules/tinyexec/package.json
A  node_modules/tinyglobby/LICENSE
A  node_modules/tinyglobby/README.md
A  node_modules/tinyglobby/dist/index.cjs
A  node_modules/tinyglobby/dist/index.d.cts
A  node_modules/tinyglobby/dist/index.d.mts
A  node_modules/tinyglobby/dist/index.mjs
A  node_modules/tinyglobby/package.json
A  node_modules/tinypool/LICENSE
A  node_modules/tinypool/README.md
A  node_modules/tinypool/dist/common-Qw-RoVFD.js
A  node_modules/tinypool/dist/entry/process.d.ts
A  node_modules/tinypool/dist/entry/process.js
A  node_modules/tinypool/dist/entry/utils.d.ts
A  node_modules/tinypool/dist/entry/utils.js
A  node_modules/tinypool/dist/entry/worker.d.ts
A  node_modules/tinypool/dist/entry/worker.js
A  node_modules/tinypool/dist/index.d.ts
A  node_modules/tinypool/dist/index.js
A  node_modules/tinypool/dist/utils-B--2TaWv.js
A  node_modules/tinypool/dist/utils-De75vAgL.js
A  node_modules/tinypool/package.json
A  node_modules/tinyrainbow/LICENCE
A  node_modules/tinyrainbow/README.md
A  node_modules/tinyrainbow/dist/browser.d.ts
A  node_modules/tinyrainbow/dist/browser.js
A  node_modules/tinyrainbow/dist/chunk-BVHSVHOK.js
A  node_modules/tinyrainbow/dist/index-8b61d5bc.d.ts
A  node_modules/tinyrainbow/dist/node.d.ts
A  node_modules/tinyrainbow/dist/node.js
A  node_modules/tinyrainbow/package.json
A  node_modules/tinyspy/LICENCE
A  node_modules/tinyspy/README.md
A  node_modules/tinyspy/dist/index.d.ts
A  node_modules/tinyspy/dist/index.js
A  node_modules/tinyspy/package.json
A  node_modules/typescript/LICENSE.txt
A  node_modules/typescript/README.md
A  node_modules/typescript/SECURITY.md
A  node_modules/typescript/ThirdPartyNoticeText.txt
A  node_modules/typescript/bin/tsc
A  node_modules/typescript/bin/tsserver
A  node_modules/typescript/lib/_tsc.js
A  node_modules/typescript/lib/_tsserver.js
A  node_modules/typescript/lib/_typingsInstaller.js
A  node_modules/typescript/lib/cs/diagnosticMessages.generated.json
A  node_modules/typescript/lib/de/diagnosticMessages.generated.json
A  node_modules/typescript/lib/es/diagnosticMessages.generated.json
A  node_modules/typescript/lib/fr/diagnosticMessages.generated.json
A  node_modules/typescript/lib/it/diagnosticMessages.generated.json
A  node_modules/typescript/lib/ja/diagnosticMessages.generated.json
A  node_modules/typescript/lib/ko/diagnosticMessages.generated.json
A  node_modules/typescript/lib/lib.d.ts
A  node_modules/typescript/lib/lib.decorators.d.ts
A  node_modules/typescript/lib/lib.decorators.legacy.d.ts
A  node_modules/typescript/lib/lib.dom.asynciterable.d.ts
A  node_modules/typescript/lib/lib.dom.d.ts
A  node_modules/typescript/lib/lib.dom.iterable.d.ts
A  node_modules/typescript/lib/lib.es2015.collection.d.ts
A  node_modules/typescript/lib/lib.es2015.core.d.ts
A  node_modules/typescript/lib/lib.es2015.d.ts
A  node_modules/typescript/lib/lib.es2015.generator.d.ts
A  node_modules/typescript/lib/lib.es2015.iterable.d.ts
A  node_modules/typescript/lib/lib.es2015.promise.d.ts
A  node_modules/typescript/lib/lib.es2015.proxy.d.ts
A  node_modules/typescript/lib/lib.es2015.reflect.d.ts
A  node_modules/typescript/lib/lib.es2015.symbol.d.ts
A  node_modules/typescript/lib/lib.es2015.symbol.wellknown.d.ts
A  node_modules/typescript/lib/lib.es2016.array.include.d.ts
A  node_modules/typescript/lib/lib.es2016.d.ts
A  node_modules/typescript/lib/lib.es2016.full.d.ts
A  node_modules/typescript/lib/lib.es2016.intl.d.ts
A  node_modules/typescript/lib/lib.es2017.arraybuffer.d.ts
A  node_modules/typescript/lib/lib.es2017.d.ts
A  node_modules/typescript/lib/lib.es2017.date.d.ts
A  node_modules/typescript/lib/lib.es2017.full.d.ts
A  node_modules/typescript/lib/lib.es2017.intl.d.ts
A  node_modules/typescript/lib/lib.es2017.object.d.ts
A  node_modules/typescript/lib/lib.es2017.sharedmemory.d.ts
A  node_modules/typescript/lib/lib.es2017.string.d.ts
A  node_modules/typescript/lib/lib.es2017.typedarrays.d.ts
A  node_modules/typescript/lib/lib.es2018.asyncgenerator.d.ts
A  node_modules/typescript/lib/lib.es2018.asynciterable.d.ts
A  node_modules/typescript/lib/lib.es2018.d.ts
A  node_modules/typescript/lib/lib.es2018.full.d.ts
A  node_modules/typescript/lib/lib.es2018.intl.d.ts
A  node_modules/typescript/lib/lib.es2018.promise.d.ts
A  node_modules/typescript/lib/lib.es2018.regexp.d.ts
A  node_modules/typescript/lib/lib.es2019.array.d.ts
A  node_modules/typescript/lib/lib.es2019.d.ts
A  node_modules/typescript/lib/lib.es2019.full.d.ts
A  node_modules/typescript/lib/lib.es2019.intl.d.ts
A  node_modules/typescript/lib/lib.es2019.object.d.ts
A  node_modules/typescript/lib/lib.es2019.string.d.ts
A  node_modules/typescript/lib/lib.es2019.symbol.d.ts
A  node_modules/typescript/lib/lib.es2020.bigint.d.ts
A  node_modules/typescript/lib/lib.es2020.d.ts
A  node_modules/typescript/lib/lib.es2020.date.d.ts
A  node_modules/typescript/lib/lib.es2020.full.d.ts
A  node_modules/typescript/lib/lib.es2020.intl.d.ts
A  node_modules/typescript/lib/lib.es2020.number.d.ts
A  node_modules/typescript/lib/lib.es2020.promise.d.ts
A  node_modules/typescript/lib/lib.es2020.sharedmemory.d.ts
A  node_modules/typescript/lib/lib.es2020.string.d.ts
A  node_modules/typescript/lib/lib.es2020.symbol.wellknown.d.ts
A  node_modules/typescript/lib/lib.es2021.d.ts
A  node_modules/typescript/lib/lib.es2021.full.d.ts
A  node_modules/typescript/lib/lib.es2021.intl.d.ts
A  node_modules/typescript/lib/lib.es2021.promise.d.ts
A  node_modules/typescript/lib/lib.es2021.string.d.ts
A  node_modules/typescript/lib/lib.es2021.weakref.d.ts
A  node_modules/typescript/lib/lib.es2022.array.d.ts
A  node_modules/typescript/lib/lib.es2022.d.ts
A  node_modules/typescript/lib/lib.es2022.error.d.ts
A  node_modules/typescript/lib/lib.es2022.full.d.ts
A  node_modules/typescript/lib/lib.es2022.intl.d.ts
A  node_modules/typescript/lib/lib.es2022.object.d.ts
A  node_modules/typescript/lib/lib.es2022.regexp.d.ts
A  node_modules/typescript/lib/lib.es2022.string.d.ts
A  node_modules/typescript/lib/lib.es2023.array.d.ts
A  node_modules/typescript/lib/lib.es2023.collection.d.ts
A  node_modules/typescript/lib/lib.es2023.d.ts
A  node_modules/typescript/lib/lib.es2023.full.d.ts
A  node_modules/typescript/lib/lib.es2023.intl.d.ts
A  node_modules/typescript/lib/lib.es2024.arraybuffer.d.ts
A  node_modules/typescript/lib/lib.es2024.collection.d.ts
A  node_modules/typescript/lib/lib.es2024.d.ts
A  node_modules/typescript/lib/lib.es2024.full.d.ts
A  node_modules/typescript/lib/lib.es2024.object.d.ts
A  node_modules/typescript/lib/lib.es2024.promise.d.ts
A  node_modules/typescript/lib/lib.es2024.regexp.d.ts
A  node_modules/typescript/lib/lib.es2024.sharedmemory.d.ts
A  node_modules/typescript/lib/lib.es2024.string.d.ts
A  node_modules/typescript/lib/lib.es5.d.ts
A  node_modules/typescript/lib/lib.es6.d.ts
A  node_modules/typescript/lib/lib.esnext.array.d.ts
A  node_modules/typescript/lib/lib.esnext.collection.d.ts
A  node_modules/typescript/lib/lib.esnext.d.ts
A  node_modules/typescript/lib/lib.esnext.decorators.d.ts
A  node_modules/typescript/lib/lib.esnext.disposable.d.ts
A  node_modules/typescript/lib/lib.esnext.error.d.ts
A  node_modules/typescript/lib/lib.esnext.float16.d.ts
A  node_modules/typescript/lib/lib.esnext.full.d.ts
A  node_modules/typescript/lib/lib.esnext.intl.d.ts
A  node_modules/typescript/lib/lib.esnext.iterator.d.ts
A  node_modules/typescript/lib/lib.esnext.promise.d.ts
A  node_modules/typescript/lib/lib.esnext.sharedmemory.d.ts
A  node_modules/typescript/lib/lib.scripthost.d.ts
A  node_modules/typescript/lib/lib.webworker.asynciterable.d.ts
A  node_modules/typescript/lib/lib.webworker.d.ts
A  node_modules/typescript/lib/lib.webworker.importscripts.d.ts
A  node_modules/typescript/lib/lib.webworker.iterable.d.ts
A  node_modules/typescript/lib/pl/diagnosticMessages.generated.json
A  node_modules/typescript/lib/pt-br/diagnosticMessages.generated.json
A  node_modules/typescript/lib/ru/diagnosticMessages.generated.json
A  node_modules/typescript/lib/tr/diagnosticMessages.generated.json
A  node_modules/typescript/lib/tsc.js
A  node_modules/typescript/lib/tsserver.js
A  node_modules/typescript/lib/tsserverlibrary.d.ts
A  node_modules/typescript/lib/tsserverlibrary.js
A  node_modules/typescript/lib/typesMap.json
A  node_modules/typescript/lib/typescript.d.ts
A  node_modules/typescript/lib/typescript.js
A  node_modules/typescript/lib/typingsInstaller.js
A  node_modules/typescript/lib/watchGuard.js
A  node_modules/typescript/lib/zh-cn/diagnosticMessages.generated.json
A  node_modules/typescript/lib/zh-tw/diagnosticMessages.generated.json
A  node_modules/typescript/package.json
A  node_modules/vite-node/LICENSE
A  node_modules/vite-node/README.md
A  node_modules/vite-node/dist/chunk-browser.cjs
A  node_modules/vite-node/dist/chunk-browser.mjs
A  node_modules/vite-node/dist/chunk-hmr.cjs
A  node_modules/vite-node/dist/chunk-hmr.mjs
A  node_modules/vite-node/dist/cli.cjs
A  node_modules/vite-node/dist/cli.d.ts
A  node_modules/vite-node/dist/cli.mjs
A  node_modules/vite-node/dist/client.cjs
A  node_modules/vite-node/dist/client.d.ts
A  node_modules/vite-node/dist/client.mjs
A  node_modules/vite-node/dist/constants.cjs
A  node_modules/vite-node/dist/constants.d.ts
A  node_modules/vite-node/dist/constants.mjs
A  node_modules/vite-node/dist/hmr.cjs
A  node_modules/vite-node/dist/hmr.d.ts
A  node_modules/vite-node/dist/hmr.mjs
A  node_modules/vite-node/dist/index.cjs
A  node_modules/vite-node/dist/index.d-DGmxD2U7.d.ts
A  node_modules/vite-node/dist/index.d.ts
A  node_modules/vite-node/dist/index.mjs
A  node_modules/vite-node/dist/server.cjs
A  node_modules/vite-node/dist/server.d.ts
A  node_modules/vite-node/dist/server.mjs
A  node_modules/vite-node/dist/source-map.cjs
A  node_modules/vite-node/dist/source-map.d.ts
A  node_modules/vite-node/dist/source-map.mjs
A  node_modules/vite-node/dist/trace-mapping.d-DLVdEqOp.d.ts
A  node_modules/vite-node/dist/types.cjs
A  node_modules/vite-node/dist/types.d.ts
A  node_modules/vite-node/dist/types.mjs
A  node_modules/vite-node/dist/utils.cjs
A  node_modules/vite-node/dist/utils.d.ts
A  node_modules/vite-node/dist/utils.mjs
A  node_modules/vite-node/package.json
A  node_modules/vite-node/vite-node.mjs
A  node_modules/vite/LICENSE.md
A  node_modules/vite/README.md
A  node_modules/vite/bin/openChrome.js
A  node_modules/vite/bin/vite.js
A  node_modules/vite/client.d.ts
A  node_modules/vite/dist/client/client.mjs
A  node_modules/vite/dist/client/env.mjs
A  node_modules/vite/dist/node/chunks/build.js
A  node_modules/vite/dist/node/chunks/build2.js
A  node_modules/vite/dist/node/chunks/chunk.js
A  node_modules/vite/dist/node/chunks/config.js
A  node_modules/vite/dist/node/chunks/config2.js
A  node_modules/vite/dist/node/chunks/dist.js
A  node_modules/vite/dist/node/chunks/lib.js
A  node_modules/vite/dist/node/chunks/logger.js
A  node_modules/vite/dist/node/chunks/moduleRunnerTransport.d.ts
A  node_modules/vite/dist/node/chunks/optimizer.js
A  node_modules/vite/dist/node/chunks/postcss-import.js
A  node_modules/vite/dist/node/chunks/preview.js
A  node_modules/vite/dist/node/chunks/server.js
A  node_modules/vite/dist/node/cli.js
A  node_modules/vite/dist/node/index.d.ts
A  node_modules/vite/dist/node/index.js
A  node_modules/vite/dist/node/module-runner.d.ts
A  node_modules/vite/dist/node/module-runner.js
A  node_modules/vite/misc/false.js
A  node_modules/vite/misc/true.js
A  node_modules/vite/package.json
A  node_modules/vite/types/customEvent.d.ts
A  node_modules/vite/types/hmrPayload.d.ts
A  node_modules/vite/types/hot.d.ts
A  node_modules/vite/types/import-meta.d.ts
A  node_modules/vite/types/importGlob.d.ts
A  node_modules/vite/types/importMeta.d.ts
A  node_modules/vite/types/internal/cssPreprocessorOptions.d.ts
A  node_modules/vite/types/internal/lightningcssOptions.d.ts
A  node_modules/vite/types/internal/terserOptions.d.ts
A  node_modules/vite/types/metadata.d.ts
A  node_modules/vite/types/package.json
A  node_modules/vitest/LICENSE.md
A  node_modules/vitest/README.md
A  node_modules/vitest/browser.d.ts
A  node_modules/vitest/config.d.ts
A  node_modules/vitest/coverage.d.ts
A  node_modules/vitest/dist/browser.d.ts
A  node_modules/vitest/dist/browser.js
A  node_modules/vitest/dist/chunks/_commonjsHelpers.BFTU3MAI.js
A  node_modules/vitest/dist/chunks/base.DfmxU-tU.js
A  node_modules/vitest/dist/chunks/benchmark.CYdenmiT.js
A  node_modules/vitest/dist/chunks/benchmark.d.BwvBVTda.d.ts
A  node_modules/vitest/dist/chunks/cac.Cb-PYCCB.js
A  node_modules/vitest/dist/chunks/cli-api.BkDphVBG.js
A  node_modules/vitest/dist/chunks/config.d.D2ROskhv.d.ts
A  node_modules/vitest/dist/chunks/console.CtFJOzRO.js
A  node_modules/vitest/dist/chunks/constants.DnKduX2e.js
A  node_modules/vitest/dist/chunks/coverage.DL5VHqXY.js
A  node_modules/vitest/dist/chunks/coverage.DVF1vEu8.js
A  node_modules/vitest/dist/chunks/coverage.d.S9RMNXIe.d.ts
A  node_modules/vitest/dist/chunks/creator.GK6I-cL4.js
A  node_modules/vitest/dist/chunks/date.Bq6ZW5rf.js
A  node_modules/vitest/dist/chunks/defaults.B7q_naMc.js
A  node_modules/vitest/dist/chunks/env.D4Lgay0q.js
A  node_modules/vitest/dist/chunks/environment.d.cL3nLXbE.d.ts
A  node_modules/vitest/dist/chunks/execute.B7h3T_Hc.js
A  node_modules/vitest/dist/chunks/git.BVQ8w_Sw.js
A  node_modules/vitest/dist/chunks/global.d.MAmajcmJ.d.ts
A  node_modules/vitest/dist/chunks/globals.DEHgCU4V.js
A  node_modules/vitest/dist/chunks/index.B521nVV-.js
A  node_modules/vitest/dist/chunks/index.BCWujgDG.js
A  node_modules/vitest/dist/chunks/index.CdQS2e2Q.js
A  node_modules/vitest/dist/chunks/index.CmSc2RE5.js
A  node_modules/vitest/dist/chunks/index.CwejwG0H.js
A  node_modules/vitest/dist/chunks/index.D3XRDfWc.js
A  node_modules/vitest/dist/chunks/index.VByaPkjc.js
A  node_modules/vitest/dist/chunks/index.X0nbfr6-.js
A  node_modules/vitest/dist/chunks/inspector.C914Efll.js
A  node_modules/vitest/dist/chunks/mocker.d.BE_2ls6u.d.ts
A  node_modules/vitest/dist/chunks/node.fjCdwEIl.js
A  node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts
A  node_modules/vitest/dist/chunks/rpc.-pEldfrD.js
A  node_modules/vitest/dist/chunks/runBaseTests.9Ij9_de-.js
A  node_modules/vitest/dist/chunks/setup-common.Dd054P77.js
A  node_modules/vitest/dist/chunks/suite.d.FvehnV49.d.ts
A  node_modules/vitest/dist/chunks/typechecker.DRKU1-1g.js
A  node_modules/vitest/dist/chunks/utils.CAioKnHs.js
A  node_modules/vitest/dist/chunks/utils.XdZDrNZV.js
A  node_modules/vitest/dist/chunks/vi.bdSIJ99Y.js
A  node_modules/vitest/dist/chunks/vite.d.CMLlLIFP.d.ts
A  node_modules/vitest/dist/chunks/vm.BThCzidc.js
A  node_modules/vitest/dist/chunks/worker.d.1GmBbd7G.d.ts
A  node_modules/vitest/dist/chunks/worker.d.CKwWzBSj.d.ts
A  node_modules/vitest/dist/cli.js
A  node_modules/vitest/dist/config.cjs
A  node_modules/vitest/dist/config.d.ts
A  node_modules/vitest/dist/config.js
A  node_modules/vitest/dist/coverage.d.ts
A  node_modules/vitest/dist/coverage.js
A  node_modules/vitest/dist/environments.d.ts
A  node_modules/vitest/dist/environments.js
A  node_modules/vitest/dist/execute.d.ts
A  node_modules/vitest/dist/execute.js
A  node_modules/vitest/dist/index.d.ts
A  node_modules/vitest/dist/index.js
A  node_modules/vitest/dist/mocker.d.ts
A  node_modules/vitest/dist/mocker.js
A  node_modules/vitest/dist/node.d.ts
A  node_modules/vitest/dist/node.js
A  node_modules/vitest/dist/path.js
A  node_modules/vitest/dist/reporters.d.ts
A  node_modules/vitest/dist/reporters.js
A  node_modules/vitest/dist/runners.d.ts
A  node_modules/vitest/dist/runners.js
A  node_modules/vitest/dist/snapshot.d.ts
A  node_modules/vitest/dist/snapshot.js
A  node_modules/vitest/dist/spy.js
A  node_modules/vitest/dist/suite.d.ts
A  node_modules/vitest/dist/suite.js
A  node_modules/vitest/dist/worker.js
A  node_modules/vitest/dist/workers.d.ts
A  node_modules/vitest/dist/workers.js
A  node_modules/vitest/dist/workers/forks.js
A  node_modules/vitest/dist/workers/runVmTests.js
A  node_modules/vitest/dist/workers/threads.js
A  node_modules/vitest/dist/workers/vmForks.js
A  node_modules/vitest/dist/workers/vmThreads.js
A  node_modules/vitest/environments.d.ts
A  node_modules/vitest/execute.d.ts
A  node_modules/vitest/globals.d.ts
A  node_modules/vitest/import-meta.d.ts
A  node_modules/vitest/importMeta.d.ts
A  node_modules/vitest/index.cjs
A  node_modules/vitest/index.d.cts
A  node_modules/vitest/jsdom.d.ts
A  node_modules/vitest/mocker.d.ts
A  node_modules/vitest/node.d.ts
A  node_modules/vitest/optional-types.d.ts
A  node_modules/vitest/package.json
A  node_modules/vitest/reporters.d.ts
A  node_modules/vitest/runners.d.ts
A  node_modules/vitest/snapshot.d.ts
A  node_modules/vitest/suite.d.ts
A  node_modules/vitest/suppress-warnings.cjs
A  node_modules/vitest/utils.d.ts
A  node_modules/vitest/vitest.mjs
A  node_modules/vitest/workers.d.ts
A  node_modules/why-is-node-running/.github/FUNDING.yml
A  node_modules/why-is-node-running/LICENSE
A  node_modules/why-is-node-running/README.md
A  node_modules/why-is-node-running/cli.js
A  node_modules/why-is-node-running/example.js
A  node_modules/why-is-node-running/include.js
A  node_modules/why-is-node-running/index.js
A  node_modules/why-is-node-running/package.json
A  package-lock.json
AM prompt.md


@[assistant]:


@[toolResult]:
(no output)

@[assistant]:
I ran a repository-wide search for both occurrences:

- `Task` interface consumers:  
  - `src/utils/format-task.ts`  
  - `src/services/task-service.ts` (defines in-memory store, `addTask`, `listTasks`)  
  - `src/models/task.ts` (definition itself)  
  - `src/reports/summary.ts`

- `formatTask` function consumers:  
  - `src/api/index.ts` (API responses)  
  - `src/services/task-service.ts` (returns formatted tasks)  
  - `src/reports/summary.ts` (summary report)  
  - `src/cli/index.ts` (CLI output)

Those are all the references in the codebase (`rg -n "Task"` / `rg -n "formatTask"`). Review these files to ensure each reflects your interface and formatting changes. No other TypeScript files reference either symbol, so as long as the above are updated, the blast-radius check is clear.