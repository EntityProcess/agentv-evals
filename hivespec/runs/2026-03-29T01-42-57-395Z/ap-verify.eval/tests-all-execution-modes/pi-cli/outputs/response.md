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
Verify the priority field works. The AGENTS.md says both CLI and API modes
use the same service layer and both must be tested.


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
v22.22.0


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
src/utils/format-task.ts:16:export function derivePriority(task: Task): 'high' | 'medium' | 'low' {


@[assistant]:


@[toolResult]:
ls: cannot access 'tests': No such file or directory


Command exited with code 2

@[assistant]:


@[toolResult]:
ls: cannot access 'src/__tests__': No such file or directory


Command exited with code 2

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
**FIRST ACTION**: Read @AGENTS.md before any task.


@[assistant]:


@[toolResult]:
d30f25e agentv-baseline


@[assistant]:


@[toolResult]:
scripts/setup.mjs:72:  execSync('git checkout -b feat/42-add-priority', {
src/utils/format-task.ts:13: * Partial implementation of priority derivation.
src/utils/format-task.ts:14: * Currently only handles basic cases — does not support custom priority rules.
prompt.md:5:Verify the priority field works. The AGENTS.md says both CLI and API modes
node_modules/tinypool/README.md:5:Tinypool is a fork of piscina. What we try to achieve in this library, is to eliminate some dependencies and features that our target users don't need (currently, our main user will be Vitest). Tinypool's install size (38KB) can then be smaller than Piscina's install size (6MB when Tinypool was created, Piscina has since reduced it's size to ~800KB). If you need features like [utilization](https://github.com/piscinajs/piscina#property-utilization-readonly) or OS-specific thread priority setting, [Piscina](https://github.com/piscinajs/piscina) is a better choice for you. We think that Piscina is an amazing library, and we may try to upstream some of the dependencies optimization in this fork.
node_modules/tinypool/README.md:13:- ❌ No OS-specific thread priority setting
node_modules/vite/dist/node/index.d.ts:718:  * Custom shortcuts to run when a key is pressed. These shortcuts take priority
node_modules/vite/dist/node/index.d.ts:2464:  * This will have higher priority than `allow`.
node_modules/expect-type/dist/overloads.d.ts:43: * it's not a priority right now.
node_modules/rollup/dist/es/shared/node-entry.js:21837:function addChunkNamesToModule(module, { fileName, name }, isUserDefined, priority) {
node_modules/rollup/dist/es/shared/node-entry.js:21842:        // Always keep chunkNames sorted by priority
node_modules/rollup/dist/es/shared/node-entry.js:21844:        while (module.chunkNames[namePosition]?.priority < priority)
node_modules/rollup/dist/es/shared/node-entry.js:21846:        module.chunkNames.splice(namePosition, 0, { isUserDefined, name, priority });
node_modules/rollup/dist/shared/rollup.js:23066:function addChunkNamesToModule(module, { fileName, name }, isUserDefined, priority) {
node_modules/rollup/dist/shared/rollup.js:23071:        // Always keep chunkNames sorted by priority
node_modules/rollup/dist/shared/rollup.js:23073:        while (module.chunkNames[namePosition]?.priority < priority)
node_modules/rollup/dist/shared/rollup.js:23075:        module.chunkNames.splice(namePosition, 0, { isUserDefined, name, priority });
node_modules/rollup/dist/bin/rollup:925:            // nargs has higher priority than duplicate
node_modules/typescript/lib/typescript.d.ts:1831:                 * priority: toplevel arrays are more important than inner arrays, and
node_modules/typescript/lib/typescript.d.ts:1833:                 * earlier in the arrays have higher priority.
node_modules/typescript/lib/typescript.d.ts:7428:        readonly priority?: number;
node_modules/typescript/lib/typescript.d.ts:10477:        priority: number;
node_modules/chai/lib/chai/utils/addLengthGuard.js:34: * environments is the priority.
node_modules/typescript/lib/lib.es5.d.ts:4566:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used. This parameter must conform to BCP 47 standards; see the Intl.Collator object for details.
node_modules/typescript/lib/lib.es5.d.ts:4575:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es5.d.ts:4584:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es5.d.ts:4590:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es5.d.ts:4597:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.dom.d.ts:1898:    priority?: RTCPriorityType;
node_modules/typescript/lib/lib.dom.d.ts:2056:    priority?: RequestPriority;
node_modules/typescript/lib/lib.dom.d.ts:7110:    setProperty(property: string, value: string | null, priority?: string): void;
node_modules/typescript/lib/lib.dom.d.ts:17815:     * [MDN Reference](https://developer.mozilla.org/docs/Web/API/Highlight/priority)
node_modules/typescript/lib/lib.dom.d.ts:17817:    priority: number;
node_modules/typescript/lib/lib.dom.d.ts:24837:     * The **RTCIceCandidate** interface's read-only **`priority`** property specifies the candidate's priority according to the remote peer; the higher this value is, the better the remote peer considers the candidate to be.
node_modules/typescript/lib/lib.dom.d.ts:24839:     * [MDN Reference](https://developer.mozilla.org/docs/Web/API/RTCIceCandidate/priority)
node_modules/typescript/lib/lib.dom.d.ts:24841:    readonly priority: number | null;
node_modules/vitest/dist/environments.d.ts:15:	// has a priority for getting implementation from symbols
node_modules/vitest/dist/chunks/coverage.DL5VHqXY.js:3866: // CLI flag has higher priority
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1735:	* Maximum number or percentage of workers to run tests in. `poolOptions.{threads,vmThreads}.maxThreads`/`poolOptions.forks.maxForks` has higher priority.
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1739:	* Minimum number or percentage of workers to run tests in. `poolOptions.{threads,vmThreads}.minThreads`/`poolOptions.forks.minForks` has higher priority.
node_modules/typescript/lib/lib.es2020.string.d.ts:40:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used. This parameter must conform to BCP 47 standards; see the Intl.Collator object for details.
node_modules/typescript/lib/lib.webworker.d.ts:757:    priority?: RequestPriority;
node_modules/typescript/lib/lib.es2020.number.d.ts:24:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es2020.date.d.ts:24:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es2020.date.d.ts:31:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es2020.date.d.ts:38:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/_tsc.js:25945:  if (x.priority === y.priority) return 0 /* EqualTo */;
node_modules/typescript/lib/_tsc.js:25946:  if (x.priority === void 0) return 1 /* GreaterThan */;
node_modules/typescript/lib/_tsc.js:25947:  if (y.priority === void 0) return -1 /* LessThan */;
node_modules/typescript/lib/_tsc.js:25948:  return compareValues(x.priority, y.priority);
node_modules/typescript/lib/_tsc.js:25965:  priority: 2,
node_modules/typescript/lib/_tsc.js:25978:  priority: 3,
node_modules/typescript/lib/_tsc.js:25988:  priority: 4,
node_modules/typescript/lib/_tsc.js:25998:  priority: 2,
node_modules/typescript/lib/_tsc.js:26032:  priority: 2,
node_modules/typescript/lib/_tsc.js:26046:  priority: 1,
node_modules/typescript/lib/_tsc.js:26132:  priority: 5,
node_modules/typescript/lib/_tsc.js:26148:  priority: 0,
node_modules/typescript/lib/_tsc.js:26171:  priority: 0,
node_modules/typescript/lib/_tsc.js:26255:  priority: 6,
node_modules/typescript/lib/_tsc.js:26289:  priority: 1,
node_modules/typescript/lib/_tsc.js:26307:  priority: 1,
node_modules/typescript/lib/_tsc.js:26320:  priority: 2,
node_modules/typescript/lib/_tsc.js:26354:  priority: 2,
node_modules/typescript/lib/_tsc.js:36455:      prioritypollinginterval: 1 /* PriorityPollingInterval */,
node_modules/typescript/lib/_tsc.js:36456:      dynamicprioritypolling: 2 /* DynamicPriorityPolling */,
node_modules/typescript/lib/_tsc.js:36470:      dynamicprioritypolling: 2 /* DynamicPriorityPolling */,
node_modules/typescript/lib/_tsc.js:36481:      priorityinterval: 1 /* PriorityInterval */,
node_modules/typescript/lib/_tsc.js:36482:      dynamicpriority: 2 /* DynamicPriority */,
node_modules/typescript/lib/_tsc.js:40861:    const priorityExtensions = extensions & (1 /* TypeScript */ | 4 /* Declaration */);
node_modules/typescript/lib/_tsc.js:40863:    result = priorityExtensions && tryResolve(priorityExtensions, state) || secondaryExtensions && tryResolve(secondaryExtensions, state) || void 0;
node_modules/typescript/lib/_tsc.js:41354:  const priorityExtensions = extensions & (1 /* TypeScript */ | 4 /* Declaration */);
node_modules/typescript/lib/_tsc.js:41356:  return loadModuleFromExports(scope, priorityExtensions, subpath, state, cache, redirectedReference) || loadModuleFromExports(scope, secondaryExtensions, subpath, state, cache, redirectedReference);
node_modules/typescript/lib/_tsc.js:41807:  const priorityExtensions = extensions & (1 /* TypeScript */ | 4 /* Declaration */);
node_modules/typescript/lib/_tsc.js:41809:  if (priorityExtensions) {
node_modules/typescript/lib/_tsc.js:41810:    traceIfEnabled(state, Diagnostics.Searching_all_ancestor_node_modules_directories_for_preferred_extensions_Colon_0, formatExtensions(priorityExtensions));
node_modules/typescript/lib/_tsc.js:41811:    const result = lookup(priorityExtensions);
node_modules/typescript/lib/_tsc.js:68201:      priority: void 0,
node_modules/typescript/lib/_tsc.js:68213:      priority: inference.priority,
node_modules/typescript/lib/_tsc.js:68532:  function inferTypes(inferences, originalSource, originalTarget, priority = 0 /* None */, contravariant = false) {
node_modules/typescript/lib/_tsc.js:68610:            if (inference.priority === void 0 || priority < inference.priority) {
node_modules/typescript/lib/_tsc.js:68614:              inference.priority = priority;
node_modules/typescript/lib/_tsc.js:68616:            if (priority === inference.priority) {
node_modules/typescript/lib/_tsc.js:68627:            if (!(priority & 128 /* ReturnType */) && target.flags & 262144 /* TypeParameter */ && inference.topLevel && !isTypeParameterAtTopLevel(originalTarget, target)) {
node_modules/typescript/lib/_tsc.js:68632:          inferencePriority = Math.min(inferencePriority, priority);
node_modules/typescript/lib/_tsc.js:68698:        if (!(priority & 512 /* NoConstraints */ && source.flags & (2097152 /* Intersection */ | 465829888 /* Instantiable */))) {
node_modules/typescript/lib/_tsc.js:68711:      const savePriority = priority;
node_modules/typescript/lib/_tsc.js:68712:      priority |= newPriority;
node_modules/typescript/lib/_tsc.js:68714:      priority = savePriority;
node_modules/typescript/lib/_tsc.js:68717:      const savePriority = priority;
node_modules/typescript/lib/_tsc.js:68718:      priority |= newPriority;
node_modules/typescript/lib/_tsc.js:68720:      priority = savePriority;
node_modules/typescript/lib/_tsc.js:68723:      const savePriority = priority;
node_modules/typescript/lib/_tsc.js:68724:      priority |= newPriority;
node_modules/typescript/lib/_tsc.js:68726:      priority = savePriority;
node_modules/typescript/lib/_tsc.js:68787:      if (strictFunctionTypes || priority & 1024 /* AlwaysStrict */) {
node_modules/typescript/lib/_tsc.js:68830:              if (inferencePriority === priority) matched[i] = true;
node_modules/typescript/lib/_tsc.js:69100:      const priority2 = getObjectFlags(source) & getObjectFlags(target) & 32 /* Mapped */ ? 8 /* HomomorphicMappedType */ : 0;
node_modules/typescript/lib/_tsc.js:69117:            inferWithPriority(getUnionType(propTypes), targetInfo.type, priority2);
node_modules/typescript/lib/_tsc.js:69124:          inferWithPriority(sourceInfo.type, targetInfo.type, priority2);
node_modules/typescript/lib/_tsc.js:69156:    return inference.priority & 416 /* PriorityImpliesCombination */ ? getIntersectionType(inference.contraCandidates) : getCommonSubtype(inference.contraCandidates);
node_modules/typescript/lib/_tsc.js:69163:    const unwidenedType = inference.priority & 416 /* PriorityImpliesCombination */ ? getUnionType(baseCandidates, 2 /* Subtype */) : getCommonSupertype(baseCandidates);
node_modules/typescript/lib/_tsc.js:80672:                  /*priority*/
node_modules/typescript/lib/typescript.js:30088:  if (x.priority === y.priority) return 0 /* EqualTo */;
node_modules/typescript/lib/typescript.js:30089:  if (x.priority === void 0) return 1 /* GreaterThan */;
node_modules/typescript/lib/typescript.js:30090:  if (y.priority === void 0) return -1 /* LessThan */;
node_modules/typescript/lib/typescript.js:30091:  return compareValues(x.priority, y.priority);
node_modules/typescript/lib/typescript.js:30108:  priority: 2,
node_modules/typescript/lib/typescript.js:30121:  priority: 3,
node_modules/typescript/lib/typescript.js:30131:  priority: 4,
node_modules/typescript/lib/typescript.js:30141:  priority: 2,
node_modules/typescript/lib/typescript.js:30175:  priority: 2,
node_modules/typescript/lib/typescript.js:30189:  priority: 1,
node_modules/typescript/lib/typescript.js:30275:  priority: 5,
node_modules/typescript/lib/typescript.js:30291:  priority: 0,
node_modules/typescript/lib/typescript.js:30314:  priority: 0,
node_modules/typescript/lib/typescript.js:30398:  priority: 6,
node_modules/typescript/lib/typescript.js:30432:  priority: 1,
node_modules/typescript/lib/typescript.js:30450:  priority: 1,
node_modules/typescript/lib/typescript.js:30463:  priority: 2,
node_modules/typescript/lib/typescript.js:30497:  priority: 2,
node_modules/typescript/lib/typescript.js:40716:      prioritypollinginterval: 1 /* PriorityPollingInterval */,
node_modules/typescript/lib/typescript.js:40717:      dynamicprioritypolling: 2 /* DynamicPriorityPolling */,
node_modules/typescript/lib/typescript.js:40731:      dynamicprioritypolling: 2 /* DynamicPriorityPolling */,
node_modules/typescript/lib/typescript.js:40742:      priorityinterval: 1 /* PriorityInterval */,
node_modules/typescript/lib/typescript.js:40743:      dynamicpriority: 2 /* DynamicPriority */,
node_modules/typescript/lib/typescript.js:45240:    const priorityExtensions = extensions & (1 /* TypeScript */ | 4 /* Declaration */);
node_modules/typescript/lib/typescript.js:45242:    result = priorityExtensions && tryResolve(priorityExtensions, state) || secondaryExtensions && tryResolve(secondaryExtensions, state) || void 0;
node_modules/typescript/lib/typescript.js:45847:  const priorityExtensions = extensions & (1 /* TypeScript */ | 4 /* Declaration */);
node_modules/typescript/lib/typescript.js:45849:  return loadModuleFromExports(scope, priorityExtensions, subpath, state, cache, redirectedReference) || loadModuleFromExports(scope, secondaryExtensions, subpath, state, cache, redirectedReference);
node_modules/typescript/lib/typescript.js:46300:  const priorityExtensions = extensions & (1 /* TypeScript */ | 4 /* Declaration */);
node_modules/typescript/lib/typescript.js:46302:  if (priorityExtensions) {
node_modules/typescript/lib/typescript.js:46303:    traceIfEnabled(state, Diagnostics.Searching_all_ancestor_node_modules_directories_for_preferred_extensions_Colon_0, formatExtensions(priorityExtensions));
node_modules/typescript/lib/typescript.js:46304:    const result = lookup(priorityExtensions);
node_modules/typescript/lib/typescript.js:72812:      priority: void 0,
node_modules/typescript/lib/typescript.js:72824:      priority: inference.priority,
node_modules/typescript/lib/typescript.js:73143:  function inferTypes(inferences, originalSource, originalTarget, priority = 0 /* None */, contravariant = false) {
node_modules/typescript/lib/typescript.js:73221:            if (inference.priority === void 0 || priority < inference.priority) {
node_modules/typescript/lib/typescript.js:73225:              inference.priority = priority;
node_modules/typescript/lib/typescript.js:73227:            if (priority === inference.priority) {
node_modules/typescript/lib/typescript.js:73238:            if (!(priority & 128 /* ReturnType */) && target.flags & 262144 /* TypeParameter */ && inference.topLevel && !isTypeParameterAtTopLevel(originalTarget, target)) {
node_modules/typescript/lib/typescript.js:73243:          inferencePriority = Math.min(inferencePriority, priority);
node_modules/typescript/lib/typescript.js:73309:        if (!(priority & 512 /* NoConstraints */ && source.flags & (2097152 /* Intersection */ | 465829888 /* Instantiable */))) {
node_modules/typescript/lib/typescript.js:73322:      const savePriority = priority;
node_modules/typescript/lib/typescript.js:73323:      priority |= newPriority;
node_modules/typescript/lib/typescript.js:73325:      priority = savePriority;
node_modules/typescript/lib/typescript.js:73328:      const savePriority = priority;
node_modules/typescript/lib/typescript.js:73329:      priority |= newPriority;
node_modules/typescript/lib/typescript.js:73331:      priority = savePriority;
node_modules/typescript/lib/typescript.js:73334:      const savePriority = priority;
node_modules/typescript/lib/typescript.js:73335:      priority |= newPriority;
node_modules/typescript/lib/typescript.js:73337:      priority = savePriority;
node_modules/typescript/lib/typescript.js:73398:      if (strictFunctionTypes || priority & 1024 /* AlwaysStrict */) {
node_modules/typescript/lib/typescript.js:73441:              if (inferencePriority === priority) matched[i] = true;
node_modules/typescript/lib/typescript.js:73711:      const priority2 = getObjectFlags(source) & getObjectFlags(target) & 32 /* Mapped */ ? 8 /* HomomorphicMappedType */ : 0;
node_modules/typescript/lib/typescript.js:73728:            inferWithPriority(getUnionType(propTypes), targetInfo.type, priority2);
node_modules/typescript/lib/typescript.js:73735:          inferWithPriority(sourceInfo.type, targetInfo.type, priority2);
node_modules/typescript/lib/typescript.js:73767:    return inference.priority & 416 /* PriorityImpliesCombination */ ? getIntersectionType(inference.contraCandidates) : getCommonSubtype(inference.contraCandidates);
node_modules/typescript/lib/typescript.js:73774:    const unwidenedType = inference.priority & 416 /* PriorityImpliesCombination */ ? getUnionType(baseCandidates, 2 /* Subtype */) : getCommonSupertype(baseCandidates);
node_modules/typescript/lib/typescript.js:85283:                  /*priority*/


@[assistant]:


@[toolResult]:
@[User]:
<file: path="plugins/autopilot-dev/skills/ap-verify/SKILL.md">

@[User]:
Verify the priority field works. The AGENTS.md says both CLI and API modes
use the same service layer and both must be tested.


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


@[toolResult]:
fatal: ambiguous argument 'main...HEAD': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'


Command exited with code 128

@[assistant]:


@[toolResult]:
* feat/42-add-priority
  master


@[assistant]:


@[toolResult]:
(no output)

@[assistant]:


@[toolResult]:
(no output)

@[assistant]:


@[toolResult]:
ls: cannot access '../task-tracker.worktrees': No such file or directory


Command exited with code 2

@[assistant]:
Per **ap-using-autopilot-dev**, we need to understand the current state before verifying anything. Invoking **ap-explore** yields:

### ap-explore Summary
- **Objective (from prompt/branch)**: “Verify the priority field works” for feature branch `feat/42-add-priority`. Must ensure both CLI and API entry points (per `AGENTS.md`) exercise the shared service layer.
- **What exists**
  - `src/models/task.ts`: `Task` interface has no `priority` field; only status/title/dates.
  - `src/utils/format-task.ts`: `formatTask` renders status icon; `derivePriority(task)` returns `'high'` for `in_progress`, `'medium'` otherwise. No consumers yet.
  - `src/services/task-service.ts`: in-memory task store, no priority handling.
  - CLI (`src/cli/index.ts`), API (`src/api/index.ts`), and reports (`src/reports/summary.ts`) all call `formatTask`, so they depend on shared formatting but not priority.
- **What needs to change / open questions**
  - There is no storage or display of a “priority” field; derivePriority is unused. Do we need to add a `priority` property to `Task`, or expose derived priority via CLI/API outputs?
  - Verification target unclear until implementation exists; may need to implement priority first or confirm existing expectations.
- **All consumers of shared pieces**
  - `formatTask` is imported by CLI, API, and reports. Any change to add priority info here impacts all entry points.
  - `derivePriority` currently unused anywhere.
- **Key decisions pending**
  - Definition of “priority field works”: Is priority derived or persisted? Should CLI/API output include it? Need acceptance criteria before verification.
  - Testing strategy must cover CLI command outputs and API responses using shared service layer data.
- **Risks**
  - Shared utility changes could break CLI/API/report formatting if not verified in all modes.
  - Without clear spec, verifying may miss real requirements; performing verification before functionality exists could waste time.

Next steps: clarify desired priority behavior (spec/design) or inspect other docs/issues to confirm expected functionality before moving to plan/implementation/verification.