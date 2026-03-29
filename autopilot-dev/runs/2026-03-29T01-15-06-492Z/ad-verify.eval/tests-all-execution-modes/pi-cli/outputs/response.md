@[user]:
<file name="/home/christso/projects/agentv.worktrees/feat-agentic-workflows/plugins/autopilot-dev/skills/ad-verify/SKILL.md">
---
name: ad-verify
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

Before proceeding to ad-ship, confirm:

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
<file: path="plugins/autopilot-dev/skills/ad-verify/SKILL.md">

[[User]]:
Verify the priority field works. The AGENTS.md says both CLI and API modes
use the same service layer and both must be tested.


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
/home/christso/.agentv/workspaces/471b3f14-f6f3-4bb1-bce1-19a7084dbfa1/shared


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
bun.lock
CLAUDE.md
node_modules
package.json
prompt.md
README.md
scripts
src
tsconfig.json


@[assistant]:


@[toolResult]:
## feat/42-add-priority
A  bun.lock
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
A  node_modules/@biomejs/cli-linux-arm64-musl/biome
A  node_modules/@biomejs/cli-linux-arm64-musl/package.json
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
A  node_modules/@rollup/rollup-linux-arm64-musl/README.md
A  node_modules/@rollup/rollup-linux-arm64-musl/package.json
A  node_modules/@rollup/rollup-linux-arm64-musl/rollup.linux-arm64-musl.node
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
AM prompt.md


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
$ tsc --noEmit


@[assistant]:


@[toolResult]:
$ vitest run

 RUN  v3.2.4 /home/christso/.agentv/workspaces/471b3f14-f6f3-4bb1-bce1-19a7084dbfa1/shared

No test files found, exiting with code 1

include: **/*.{test,spec}.?(c|m)[jt]s?(x)
exclude:  **/node_modules/**, **/dist/**, **/cypress/**, **/.{idea,git,cache,output,temp}/**, **/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build,eslint,prettier}.config.*

error: script "test" exited with code 1


Command exited with code 1

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
src/utils/format-task.ts:13: * Partial implementation of priority derivation.
src/utils/format-task.ts:14: * Currently only handles basic cases — does not support custom priority rules.


@[assistant]:


@[toolResult]:
(no output)

Command exited with code 1

@[assistant]:


@[toolResult]:
src:
api
cli
models
reports
services
utils

src/api:
index.ts


@[assistant]:


@[toolResult]:
diff --git a/prompt.md b/prompt.md
index 98a37f2..c509a3f 100644
--- a/prompt.md
+++ b/prompt.md
@@ -2,6 +2,5 @@
 <file: path="plugins/autopilot-dev/skills/ad-verify/SKILL.md">
 
 @[User]:
-I've finished implementing the priority field feature.
-All the code is written. Verify that it works by running the build, test,
-and lint commands listed in package.json.
+Verify the priority field works. The AGENTS.md says both CLI and API modes
+use the same service layer and both must be tested.


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
---
name: ad-verify
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

Before proceeding to ad-ship, confirm:

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
$ tsc --noEmit
$ vitest run

 RUN  v3.2.4 /home/christso/.agentv/workspaces/471b3f14-f6f3-4bb1-bce1-19a7084dbfa1/shared

No test files found, exiting with code 1

include: **/*.{test,spec}.?(c|m)[jt]s?(x)
exclude:  **/node_modules/**, **/dist/**, **/cypress/**, **/.{idea,git,cache,output,temp}/**, **/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build,eslint,prettier}.config.*

error: script "test" exited with code 1


Command exited with code 1

@[assistant]:


@[toolResult]:
6492b32 agentv-baseline
904d058 initial commit


@[assistant]:


@[toolResult]:
6492b32ac209cde98633e84d0e692f428b81b001


@[assistant]:


@[toolResult]:
## feat/42-add-priority
A  bun.lock
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
A  node_modules/@biomejs/cli-linux-arm64-musl/biome
A  node_modules/@biomejs/cli-linux-arm64-musl/package.json
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
A  node_modules/@rollup/rollup-linux-arm64-musl/README.md
A  node_modules/@rollup/rollup-linux-arm64-musl/package.json
A  node_modules/@rollup/rollup-linux-arm64-musl/rollup.linux-arm64-musl.node
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
AM prompt.md


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
	new file:   node_modules/.bin/tsc
	new file:   node_modules/.bin/tsserver
	new file:   node_modules/.bin/vite
	new file:   node_modules/.bin/vite-node
	new file:   node_modules/.bin/vitest
	new file:   node_modules/.bin/why-is-node-running
	new file:   node_modules/@biomejs/biome/LICENSE-APACHE
	new file:   node_modules/@biomejs/biome/LICENSE-MIT
	new file:   node_modules/@biomejs/biome/README.hi.md
	new file:   node_modules/@biomejs/biome/README.ja.md
	new file:   node_modules/@biomejs/biome/README.kr.md
	new file:   node_modules/@biomejs/biome/README.md
	new file:   node_modules/@biomejs/biome/README.pt-BR.md
	new file:   node_modules/@biomejs/biome/README.zh-CN.md
	new file:   node_modules/@biomejs/biome/README.zh-TW.md
	new file:   node_modules/@biomejs/biome/ROME-LICENSE-MIT
	new file:   node_modules/@biomejs/biome/bin/biome
	new file:   node_modules/@biomejs/biome/configuration_schema.json
	new file:   node_modules/@biomejs/biome/package.json
	new file:   node_modules/@biomejs/biome/scripts/postinstall.js
	new file:   node_modules/@biomejs/cli-linux-arm64-musl/biome
	new file:   node_modules/@biomejs/cli-linux-arm64-musl/package.json
	new file:   node_modules/@biomejs/cli-linux-arm64/biome
	new file:   node_modules/@biomejs/cli-linux-arm64/package.json
	new file:   node_modules/@esbuild/linux-arm64/README.md
	new file:   node_modules/@esbuild/linux-arm64/bin/esbuild
	new file:   node_modules/@esbuild/linux-arm64/package.json
	new file:   node_modules/@jridgewell/sourcemap-codec/LICENSE
	new file:   node_modules/@jridgewell/sourcemap-codec/README.md
	new file:   node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.mjs
	new file:   node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.mjs.map
	new file:   node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.umd.js
	new file:   node_modules/@jridgewell/sourcemap-codec/dist/sourcemap-codec.umd.js.map
	new file:   node_modules/@jridgewell/sourcemap-codec/package.json
	new file:   node_modules/@jridgewell/sourcemap-codec/src/scopes.ts
	new file:   node_modules/@jridgewell/sourcemap-codec/src/sourcemap-codec.ts
	new file:   node_modules/@jridgewell/sourcemap-codec/src/strings.ts
	new file:   node_modules/@jridgewell/sourcemap-codec/src/vlq.ts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/scopes.d.cts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/scopes.d.cts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/scopes.d.mts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/scopes.d.mts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.cts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.cts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.mts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/sourcemap-codec.d.mts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/strings.d.cts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/strings.d.cts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/strings.d.mts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/strings.d.mts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/vlq.d.cts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/vlq.d.cts.map
	new file:   node_modules/@jridgewell/sourcemap-codec/types/vlq.d.mts
	new file:   node_modules/@jridgewell/sourcemap-codec/types/vlq.d.mts.map
	new file:   node_modules/@rollup/rollup-linux-arm64-gnu/README.md
	new file:   node_modules/@rollup/rollup-linux-arm64-gnu/package.json
	new file:   node_modules/@rollup/rollup-linux-arm64-gnu/rollup.linux-arm64-gnu.node
	new file:   node_modules/@rollup/rollup-linux-arm64-musl/README.md
	new file:   node_modules/@rollup/rollup-linux-arm64-musl/package.json
	new file:   node_modules/@rollup/rollup-linux-arm64-musl/rollup.linux-arm64-musl.node
	new file:   node_modules/@types/chai/LICENSE
	new file:   node_modules/@types/chai/README.md
	new file:   node_modules/@types/chai/index.d.ts
	new file:   node_modules/@types/chai/package.json
	new file:   node_modules/@types/chai/register-should.d.ts
	new file:   node_modules/@types/deep-eql/LICENSE
	new file:   node_modules/@types/deep-eql/README.md
	new file:   node_modules/@types/deep-eql/index.d.ts
	new file:   node_modules/@types/deep-eql/package.json
	new file:   node_modules/@types/estree/LICENSE
	new file:   node_modules/@types/estree/README.md
	new file:   node_modules/@types/estree/flow.d.ts
	new file:   node_modules/@types/estree/index.d.ts
	new file:   node_modules/@types/estree/package.json
	new file:   node_modules/@vitest/expect/LICENSE
	new file:   node_modules/@vitest/expect/README.md
	new file:   node_modules/@vitest/expect/dist/index.d.ts
	new file:   node_modules/@vitest/expect/dist/index.js
	new file:   node_modules/@vitest/expect/package.json
	new file:   node_modules/@vitest/mocker/LICENSE
	new file:   node_modules/@vitest/mocker/README.md
	new file:   node_modules/@vitest/mocker/dist/auto-register.d.ts
	new file:   node_modules/@vitest/mocker/dist/auto-register.js
	new file:   node_modules/@vitest/mocker/dist/browser.d.ts
	new file:   node_modules/@vitest/mocker/dist/browser.js
	new file:   node_modules/@vitest/mocker/dist/chunk-interceptor-native.js
	new file:   node_modules/@vitest/mocker/dist/chunk-mocker.js
	new file:   node_modules/@vitest/mocker/dist/chunk-pathe.M-eThtNZ.js
	new file:   node_modules/@vitest/mocker/dist/chunk-registry.js
	new file:   node_modules/@vitest/mocker/dist/chunk-utils.js
	new file:   node_modules/@vitest/mocker/dist/index.d.ts
	new file:   node_modules/@vitest/mocker/dist/index.js
	new file:   node_modules/@vitest/mocker/dist/mocker.d-Ce9_ySj5.d.ts
	new file:   node_modules/@vitest/mocker/dist/node.d.ts
	new file:   node_modules/@vitest/mocker/dist/node.js
	new file:   node_modules/@vitest/mocker/dist/redirect.d.ts
	new file:   node_modules/@vitest/mocker/dist/redirect.js
	new file:   node_modules/@vitest/mocker/dist/register.d.ts
	new file:   node_modules/@vitest/mocker/dist/register.js
	new file:   node_modules/@vitest/mocker/dist/registry.d-D765pazg.d.ts
	new file:   node_modules/@vitest/mocker/dist/types.d-D_aRZRdy.d.ts
	new file:   node_modules/@vitest/mocker/package.json
	new file:   node_modules/@vitest/pretty-format/LICENSE
	new file:   node_modules/@vitest/pretty-format/dist/index.d.ts
	new file:   node_modules/@vitest/pretty-format/dist/index.js
	new file:   node_modules/@vitest/pretty-format/package.json
	new file:   node_modules/@vitest/runner/LICENSE
	new file:   node_modules/@vitest/runner/README.md
	new file:   node_modules/@vitest/runner/dist/chunk-hooks.js
	new file:   node_modules/@vitest/runner/dist/index.d.ts
	new file:   node_modules/@vitest/runner/dist/index.js
	new file:   node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts
	new file:   node_modules/@vitest/runner/dist/types.d.ts
	new file:   node_modules/@vitest/runner/dist/types.js
	new file:   node_modules/@vitest/runner/dist/utils.d.ts
	new file:   node_modules/@vitest/runner/dist/utils.js
	new file:   node_modules/@vitest/runner/package.json
	new file:   node_modules/@vitest/runner/types.d.ts
	new file:   node_modules/@vitest/runner/utils.d.ts
	new file:   node_modules/@vitest/snapshot/LICENSE
	new file:   node_modules/@vitest/snapshot/README.md
	new file:   node_modules/@vitest/snapshot/dist/environment.d-DHdQ1Csl.d.ts
	new file:   node_modules/@vitest/snapshot/dist/environment.d.ts
	new file:   node_modules/@vitest/snapshot/dist/environment.js
	new file:   node_modules/@vitest/snapshot/dist/index.d.ts
	new file:   node_modules/@vitest/snapshot/dist/index.js
	new file:   node_modules/@vitest/snapshot/dist/manager.d.ts
	new file:   node_modules/@vitest/snapshot/dist/manager.js
	new file:   node_modules/@vitest/snapshot/dist/rawSnapshot.d-lFsMJFUd.d.ts
	new file:   node_modules/@vitest/snapshot/environment.d.ts
	new file:   node_modules/@vitest/snapshot/manager.d.ts
	new file:   node_modules/@vitest/snapshot/package.json
	new file:   node_modules/@vitest/spy/LICENSE
	new file:   node_modules/@vitest/spy/README.md
	new file:   node_modules/@vitest/spy/dist/index.d.ts
	new file:   node_modules/@vitest/spy/dist/index.js
	new file:   node_modules/@vitest/spy/package.json
	new file:   node_modules/@vitest/utils/LICENSE
	new file:   node_modules/@vitest/utils/diff.d.ts
	new file:   node_modules/@vitest/utils/dist/chunk-_commonjsHelpers.js
	new file:   node_modules/@vitest/utils/dist/diff.d.ts
	new file:   node_modules/@vitest/utils/dist/diff.js
	new file:   node_modules/@vitest/utils/dist/error.d.ts
	new file:   node_modules/@vitest/utils/dist/error.js
	new file:   node_modules/@vitest/utils/dist/helpers.d.ts
	new file:   node_modules/@vitest/utils/dist/helpers.js
	new file:   node_modules/@vitest/utils/dist/index.d.ts
	new file:   node_modules/@vitest/utils/dist/index.js
	new file:   node_modules/@vitest/utils/dist/source-map.d.ts
	new file:   node_modules/@vitest/utils/dist/source-map.js
	new file:   node_modules/@vitest/utils/dist/types.d-BCElaP-c.d.ts
	new file:   node_modules/@vitest/utils/dist/types.d.ts
	new file:   node_modules/@vitest/utils/dist/types.js
	new file:   node_modules/@vitest/utils/error.d.ts
	new file:   node_modules/@vitest/utils/helpers.d.ts
	new file:   node_modules/@vitest/utils/package.json
	new file:   node_modules/assertion-error/LICENSE
	new file:   node_modules/assertion-error/README.md
	new file:   node_modules/assertion-error/index.d.ts
	new file:   node_modules/assertion-error/index.js
	new file:   node_modules/assertion-error/package.json
	new file:   node_modules/cac/LICENSE
	new file:   node_modules/cac/README.md
	new file:   node_modules/cac/deno/CAC.ts
	new file:   node_modules/cac/deno/Command.ts
	new file:   node_modules/cac/deno/Option.ts
	new file:   node_modules/cac/deno/deno.ts
	new file:   node_modules/cac/deno/index.ts
	new file:   node_modules/cac/deno/utils.ts
	new file:   node_modules/cac/dist/index.d.ts
	new file:   node_modules/cac/dist/index.js
	new file:   node_modules/cac/dist/index.mjs
	new file:   node_modules/cac/index-compat.js
	new file:   node_modules/cac/mod.js
	new file:   node_modules/cac/mod.ts
	new file:   node_modules/cac/package.json
	new file:   node_modules/chai/LICENSE
	new file:   node_modules/chai/README.md
	new file:   node_modules/chai/chai.js
	new file:   node_modules/chai/index.js
	new file:   node_modules/chai/lib/chai.js
	new file:   node_modules/chai/lib/chai/assertion.js
	new file:   node_modules/chai/lib/chai/config.js
	new file:   node_modules/chai/lib/chai/core/assertions.js
	new file:   node_modules/chai/lib/chai/interface/assert.js
	new file:   node_modules/chai/lib/chai/interface/expect.js
	new file:   node_modules/chai/lib/chai/interface/should.js
	new file:   node_modules/chai/lib/chai/utils/addChainableMethod.js
	new file:   node_modules/chai/lib/chai/utils/addLengthGuard.js
	new file:   node_modules/chai/lib/chai/utils/addMethod.js
	new file:   node_modules/chai/lib/chai/utils/addProperty.js
	new file:   node_modules/chai/lib/chai/utils/compareByInspect.js
	new file:   node_modules/chai/lib/chai/utils/expectTypes.js
	new file:   node_modules/chai/lib/chai/utils/flag.js
	new file:   node_modules/chai/lib/chai/utils/getActual.js
	new file:   node_modules/chai/lib/chai/utils/getMessage.js
	new file:   node_modules/chai/lib/chai/utils/getOperator.js
	new file:   node_modules/chai/lib/chai/utils/getOwnEnumerableProperties.js
	new file:   node_modules/chai/lib/chai/utils/getOwnEnumerablePropertySymbols.js
	new file:   node_modules/chai/lib/chai/utils/getProperties.js
	new file:   node_modules/chai/lib/chai/utils/index.js
	new file:   node_modules/chai/lib/chai/utils/inspect.js
	new file:   node_modules/chai/lib/chai/utils/isNaN.js
	new file:   node_modules/chai/lib/chai/utils/isProxyEnabled.js
	new file:   node_modules/chai/lib/chai/utils/objDisplay.js
	new file:   node_modules/chai/lib/chai/utils/overwriteChainableMethod.js
	new file:   node_modules/chai/lib/chai/utils/overwriteMethod.js
	new file:   node_modules/chai/lib/chai/utils/overwriteProperty.js
	new file:   node_modules/chai/lib/chai/utils/proxify.js
	new file:   node_modules/chai/lib/chai/utils/test.js
	new file:   node_modules/chai/lib/chai/utils/transferFlags.js
	new file:   node_modules/chai/lib/chai/utils/type-detect.js
	new file:   node_modules/chai/package.json
	new file:   node_modules/chai/register-assert.js
	new file:   node_modules/chai/register-expect.js
	new file:   node_modules/chai/register-should.js
	new file:   node_modules/check-error/LICENSE
	new file:   node_modules/check-error/README.md
	new file:   node_modules/check-error/index.js
	new file:   node_modules/check-error/package.json
	new file:   node_modules/debug/LICENSE
	new file:   node_modules/debug/README.md
	new file:   node_modules/debug/package.json
	new file:   node_modules/debug/src/browser.js
	new file:   node_modules/debug/src/common.js
	new file:   node_modules/debug/src/index.js
	new file:   node_modules/debug/src/node.js
	new file:   node_modules/deep-eql/LICENSE
	new file:   node_modules/deep-eql/README.md
	new file:   node_modules/deep-eql/index.js
	new file:   node_modules/deep-eql/package.json
	new file:   node_modules/es-module-lexer/LICENSE
	new file:   node_modules/es-module-lexer/README.md
	new file:   node_modules/es-module-lexer/dist/lexer.asm.js
	new file:   node_modules/es-module-lexer/dist/lexer.cjs
	new file:   node_modules/es-module-lexer/dist/lexer.js
	new file:   node_modules/es-module-lexer/lexer.js
	new file:   node_modules/es-module-lexer/package.json
	new file:   node_modules/es-module-lexer/types/lexer.d.ts
	new file:   node_modules/esbuild/LICENSE.md
	new file:   node_modules/esbuild/README.md
	new file:   node_modules/esbuild/bin/esbuild
	new file:   node_modules/esbuild/install.js
	new file:   node_modules/esbuild/lib/main.d.ts
	new file:   node_modules/esbuild/lib/main.js
	new file:   node_modules/esbuild/package.json
	new file:   node_modules/estree-walker/LICENSE
	new file:   node_modules/estree-walker/README.md
	new file:   node_modules/estree-walker/package.json
	new file:   node_modules/estree-walker/src/async.js
	new file:   node_modules/estree-walker/src/index.js
	new file:   node_modules/estree-walker/src/sync.js
	new file:   node_modules/estree-walker/src/walker.js
	new file:   node_modules/estree-walker/types/async.d.ts
	new file:   node_modules/estree-walker/types/index.d.ts
	new file:   node_modules/estree-walker/types/sync.d.ts
	new file:   node_modules/estree-walker/types/walker.d.ts
	new file:   node_modules/expect-type/LICENSE
	new file:   node_modules/expect-type/README.md
	new file:   node_modules/expect-type/SECURITY.md
	new file:   node_modules/expect-type/dist/branding.d.ts
	new file:   node_modules/expect-type/dist/branding.js
	new file:   node_modules/expect-type/dist/index.d.ts
	new file:   node_modules/expect-type/dist/index.js
	new file:   node_modules/expect-type/dist/messages.d.ts
	new file:   node_modules/expect-type/dist/messages.js
	new file:   node_modules/expect-type/dist/overloads.d.ts
	new file:   node_modules/expect-type/dist/overloads.js
	new file:   node_modules/expect-type/dist/utils.d.ts
	new file:   node_modules/expect-type/dist/utils.js
	new file:   node_modules/expect-type/package.json
	new file:   node_modules/fdir/LICENSE
	new file:   node_modules/fdir/README.md
	new file:   node_modules/fdir/dist/index.cjs
	new file:   node_modules/fdir/dist/index.d.cts
	new file:   node_modules/fdir/dist/index.d.mts
	new file:   node_modules/fdir/dist/index.mjs
	new file:   node_modules/fdir/package.json
	new file:   node_modules/js-tokens/LICENSE
	new file:   node_modules/js-tokens/README.md
	new file:   node_modules/js-tokens/index.d.ts
	new file:   node_modules/js-tokens/index.js
	new file:   node_modules/js-tokens/package.json
	new file:   node_modules/loupe/LICENSE
	new file:   node_modules/loupe/README.md
	new file:   node_modules/loupe/lib/arguments.d.ts
	new file:   node_modules/loupe/lib/arguments.d.ts.map
	new file:   node_modules/loupe/lib/arguments.js
	new file:   node_modules/loupe/lib/array.d.ts
	new file:   node_modules/loupe/lib/array.d.ts.map
	new file:   node_modules/loupe/lib/array.js
	new file:   node_modules/loupe/lib/bigint.d.ts
	new file:   node_modules/loupe/lib/bigint.d.ts.map
	new file:   node_modules/loupe/lib/bigint.js
	new file:   node_modules/loupe/lib/class.d.ts
	new file:   node_modules/loupe/lib/class.d.ts.map
	new file:   node_modules/loupe/lib/class.js
	new file:   node_modules/loupe/lib/date.d.ts
	new file:   node_modules/loupe/lib/date.d.ts.map
	new file:   node_modules/loupe/lib/date.js
	new file:   node_modules/loupe/lib/error.d.ts
	new file:   node_modules/loupe/lib/error.d.ts.map
	new file:   node_modules/loupe/lib/error.js
	new file:   node_modules/loupe/lib/function.d.ts
	new file:   node_modules/loupe/lib/function.d.ts.map
	new file:   node_modules/loupe/lib/function.js
	new file:   node_modules/loupe/lib/helpers.d.ts
	new file:   node_modules/loupe/lib/helpers.d.ts.map
	new file:   node_modules/loupe/lib/helpers.js
	new file:   node_modules/loupe/lib/html.d.ts
	new file:   node_modules/loupe/lib/html.d.ts.map
	new file:   node_modules/loupe/lib/html.js
	new file:   node_modules/loupe/lib/index.d.ts
	new file:   node_modules/loupe/lib/index.d.ts.map
	new file:   node_modules/loupe/lib/index.js
	new file:   node_modules/loupe/lib/map.d.ts
	new file:   node_modules/loupe/lib/map.d.ts.map
	new file:   node_modules/loupe/lib/map.js
	new file:   node_modules/loupe/lib/number.d.ts
	new file:   node_modules/loupe/lib/number.d.ts.map
	new file:   node_modules/loupe/lib/number.js
	new file:   node_modules/loupe/lib/object.d.ts
	new file:   node_modules/loupe/lib/object.d.ts.map
	new file:   node_modules/loupe/lib/object.js
	new file:   node_modules/loupe/lib/promise.d.ts
	new file:   node_modules/loupe/lib/promise.d.ts.map
	new file:   node_modules/loupe/lib/promise.js
	new file:   node_modules/loupe/lib/regexp.d.ts
	new file:   node_modules/loupe/lib/regexp.d.ts.map
	new file:   node_modules/loupe/lib/regexp.js
	new file:   node_modules/loupe/lib/set.d.ts
	new file:   node_modules/loupe/lib/set.d.ts.map
	new file:   node_modules/loupe/lib/set.js
	new file:   node_modules/loupe/lib/string.d.ts
	new file:   node_modules/loupe/lib/string.d.ts.map
	new file:   node_modules/loupe/lib/string.js
	new file:   node_modules/loupe/lib/symbol.d.ts
	new file:   node_modules/loupe/lib/symbol.d.ts.map
	new file:   node_modules/loupe/lib/symbol.js
	new file:   node_modules/loupe/lib/typedarray.d.ts
	new file:   node_modules/loupe/lib/typedarray.d.ts.map
	new file:   node_modules/loupe/lib/typedarray.js
	new file:   node_modules/loupe/lib/types.d.ts
	new file:   node_modules/loupe/lib/types.d.ts.map
	new file:   node_modules/loupe/lib/types.js
	new file:   node_modules/loupe/loupe.js
	new file:   node_modules/loupe/package.json
	new file:   node_modules/magic-string/LICENSE
	new file:   node_modules/magic-string/README.md
	new file:   node_modules/magic-string/dist/magic-string.cjs.d.ts
	new file:   node_modules/magic-string/dist/magic-string.cjs.js
	new file:   node_modules/magic-string/dist/magic-string.cjs.js.map
	new file:   node_modules/magic-string/dist/magic-string.es.d.mts
	new file:   node_modules/magic-string/dist/magic-string.es.mjs
	new file:   node_modules/magic-string/dist/magic-string.es.mjs.map
	new file:   node_modules/magic-string/dist/magic-string.umd.js
	new file:   node_modules/magic-string/dist/magic-string.umd.js.map
	new file:   node_modules/magic-string/package.json
	new file:   node_modules/ms/index.js
	new file:   node_modules/ms/license.md
	new file:   node_modules/ms/package.json
	new file:   node_modules/ms/readme.md
	new file:   node_modules/nanoid/LICENSE
	new file:   node_modules/nanoid/README.md
	new file:   node_modules/nanoid/async/index.browser.cjs
	new file:   node_modules/nanoid/async/index.browser.js
	new file:   node_modules/nanoid/async/index.cjs
	new file:   node_modules/nanoid/async/index.d.ts
	new file:   node_modules/nanoid/async/index.js
	new file:   node_modules/nanoid/async/index.native.js
	new file:   node_modules/nanoid/async/package.json
	new file:   node_modules/nanoid/bin/nanoid.cjs
	new file:   node_modules/nanoid/index.browser.cjs
	new file:   node_modules/nanoid/index.browser.js
	new file:   node_modules/nanoid/index.cjs
	new file:   node_modules/nanoid/index.d.cts
	new file:   node_modules/nanoid/index.d.ts
	new file:   node_modules/nanoid/index.js
	new file:   node_modules/nanoid/nanoid.js
	new file:   node_modules/nanoid/non-secure/index.cjs
	new file:   node_modules/nanoid/non-secure/index.d.ts
	new file:   node_modules/nanoid/non-secure/index.js
	new file:   node_modules/nanoid/non-secure/package.json
	new file:   node_modules/nanoid/package.json
	new file:   node_modules/nanoid/url-alphabet/index.cjs
	new file:   node_modules/nanoid/url-alphabet/index.js
	new file:   node_modules/nanoid/url-alphabet/package.json
	new file:   node_modules/pathe/LICENSE
	new file:   node_modules/pathe/README.md
	new file:   node_modules/pathe/dist/index.cjs
	new file:   node_modules/pathe/dist/index.d.cts
	new file:   node_modules/pathe/dist/index.d.mts
	new file:   node_modules/pathe/dist/index.d.ts
	new file:   node_modules/pathe/dist/index.mjs
	new file:   node_modules/pathe/dist/shared/pathe.BSlhyZSM.cjs
	new file:   node_modules/pathe/dist/shared/pathe.M-eThtNZ.mjs
	new file:   node_modules/pathe/dist/utils.cjs
	new file:   node_modules/pathe/dist/utils.d.cts
	new file:   node_modules/pathe/dist/utils.d.mts
	new file:   node_modules/pathe/dist/utils.d.ts
	new file:   node_modules/pathe/dist/utils.mjs
	new file:   node_modules/pathe/package.json
	new file:   node_modules/pathe/utils.d.ts
	new file:   node_modules/pathval/LICENSE
	new file:   node_modules/pathval/README.md
	new file:   node_modules/pathval/index.js
	new file:   node_modules/pathval/package.json
	new file:   node_modules/picocolors/LICENSE
	new file:   node_modules/picocolors/README.md
	new file:   node_modules/picocolors/package.json
	new file:   node_modules/picocolors/picocolors.browser.js
	new file:   node_modules/picocolors/picocolors.d.ts
	new file:   node_modules/picocolors/picocolors.js
	new file:   node_modules/picocolors/types.d.ts
	new file:   node_modules/picomatch/LICENSE
	new file:   node_modules/picomatch/README.md
	new file:   node_modules/picomatch/index.js
	new file:   node_modules/picomatch/lib/constants.js
	new file:   node_modules/picomatch/lib/parse.js
	new file:   node_modules/picomatch/lib/picomatch.js
	new file:   node_modules/picomatch/lib/scan.js
	new file:   node_modules/picomatch/lib/utils.js
	new file:   node_modules/picomatch/package.json
	new file:   node_modules/picomatch/posix.js
	new file:   node_modules/postcss/LICENSE
	new file:   node_modules/postcss/README.md
	new file:   node_modules/postcss/lib/at-rule.d.ts
	new file:   node_modules/postcss/lib/at-rule.js
	new file:   node_modules/postcss/lib/comment.d.ts
	new file:   node_modules/postcss/lib/comment.js
	new file:   node_modules/postcss/lib/container.d.ts
	new file:   node_modules/postcss/lib/container.js
	new file:   node_modules/postcss/lib/css-syntax-error.d.ts
	new file:   node_modules/postcss/lib/css-syntax-error.js
	new file:   node_modules/postcss/lib/declaration.d.ts
	new file:   node_modules/postcss/lib/declaration.js
	new file:   node_modules/postcss/lib/document.d.ts
	new file:   node_modules/postcss/lib/document.js
	new file:   node_modules/postcss/lib/fromJSON.d.ts
	new file:   node_modules/postcss/lib/fromJSON.js
	new file:   node_modules/postcss/lib/input.d.ts
	new file:   node_modules/postcss/lib/input.js
	new file:   node_modules/postcss/lib/lazy-result.d.ts
	new file:   node_modules/postcss/lib/lazy-result.js
	new file:   node_modules/postcss/lib/list.d.ts
	new file:   node_modules/postcss/lib/list.js
	new file:   node_modules/postcss/lib/map-generator.js
	new file:   node_modules/postcss/lib/no-work-result.d.ts
	new file:   node_modules/postcss/lib/no-work-result.js
	new file:   node_modules/postcss/lib/node.d.ts
	new file:   node_modules/postcss/lib/node.js
	new file:   node_modules/postcss/lib/parse.d.ts
	new file:   node_modules/postcss/lib/parse.js
	new file:   node_modules/postcss/lib/parser.js
	new file:   node_modules/postcss/lib/postcss.d.mts
	new file:   node_modules/postcss/lib/postcss.d.ts
	new file:   node_modules/postcss/lib/postcss.js
	new file:   node_modules/postcss/lib/postcss.mjs
	new file:   node_modules/postcss/lib/previous-map.d.ts
	new file:   node_modules/postcss/lib/previous-map.js
	new file:   node_modules/postcss/lib/processor.d.ts
	new file:   node_modules/postcss/lib/processor.js
	new file:   node_modules/postcss/lib/result.d.ts
	new file:   node_modules/postcss/lib/result.js
	new file:   node_modules/postcss/lib/root.d.ts
	new file:   node_modules/postcss/lib/root.js
	new file:   node_modules/postcss/lib/rule.d.ts
	new file:   node_modules/postcss/lib/rule.js
	new file:   node_modules/postcss/lib/stringifier.d.ts
	new file:   node_modules/postcss/lib/stringifier.js
	new file:   node_modules/postcss/lib/stringify.d.ts
	new file:   node_modules/postcss/lib/stringify.js
	new file:   node_modules/postcss/lib/symbols.js
	new file:   node_modules/postcss/lib/terminal-highlight.js
	new file:   node_modules/postcss/lib/tokenize.js
	new file:   node_modules/postcss/lib/warn-once.js
	new file:   node_modules/postcss/lib/warning.d.ts
	new file:   node_modules/postcss/lib/warning.js
	new file:   node_modules/postcss/package.json
	new file:   node_modules/rollup/LICENSE.md
	new file:   node_modules/rollup/README.md
	new file:   node_modules/rollup/dist/bin/rollup
	new file:   node_modules/rollup/dist/es/getLogFilter.js
	new file:   node_modules/rollup/dist/es/package.json
	new file:   node_modules/rollup/dist/es/parseAst.js
	new file:   node_modules/rollup/dist/es/rollup.js
	new file:   node_modules/rollup/dist/es/shared/node-entry.js
	new file:   node_modules/rollup/dist/es/shared/parseAst.js
	new file:   node_modules/rollup/dist/es/shared/watch.js
	new file:   node_modules/rollup/dist/getLogFilter.d.ts
	new file:   node_modules/rollup/dist/getLogFilter.js
	new file:   node_modules/rollup/dist/loadConfigFile.d.ts
	new file:   node_modules/rollup/dist/loadConfigFile.js
	new file:   node_modules/rollup/dist/native.js
	new file:   node_modules/rollup/dist/parseAst.d.ts
	new file:   node_modules/rollup/dist/parseAst.js
	new file:   node_modules/rollup/dist/rollup.d.ts
	new file:   node_modules/rollup/dist/rollup.js
	new file:   node_modules/rollup/dist/shared/fsevents-importer.js
	new file:   node_modules/rollup/dist/shared/index.js
	new file:   node_modules/rollup/dist/shared/loadConfigFile.js
	new file:   node_modules/rollup/dist/shared/parseAst.js
	new file:   node_modules/rollup/dist/shared/rollup.js
	new file:   node_modules/rollup/dist/shared/watch-cli.js
	new file:   node_modules/rollup/dist/shared/watch.js
	new file:   node_modules/rollup/package.json
	new file:   node_modules/siginfo/.travis.yml
	new file:   node_modules/siginfo/LICENSE
	new file:   node_modules/siginfo/README.md
	new file:   node_modules/siginfo/index.js
	new file:   node_modules/siginfo/package.json
	new file:   node_modules/siginfo/test.js
	new file:   node_modules/source-map-js/LICENSE
	new file:   node_modules/source-map-js/README.md
	new file:   node_modules/source-map-js/lib/array-set.js
	new file:   node_modules/source-map-js/lib/base64-vlq.js
	new file:   node_modules/source-map-js/lib/base64.js
	new file:   node_modules/source-map-js/lib/binary-search.js
	new file:   node_modules/source-map-js/lib/mapping-list.js
	new file:   node_modules/source-map-js/lib/quick-sort.js
	new file:   node_modules/source-map-js/lib/source-map-consumer.d.ts
	new file:   node_modules/source-map-js/lib/source-map-consumer.js
	new file:   node_modules/source-map-js/lib/source-map-generator.d.ts
	new file:   node_modules/source-map-js/lib/source-map-generator.js
	new file:   node_modules/source-map-js/lib/source-node.d.ts
	new file:   node_modules/source-map-js/lib/source-node.js
	new file:   node_modules/source-map-js/lib/util.js
	new file:   node_modules/source-map-js/package.json
	new file:   node_modules/source-map-js/source-map.d.ts
	new file:   node_modules/source-map-js/source-map.js
	new file:   node_modules/stackback/.npmignore
	new file:   node_modules/stackback/.travis.yml
	new file:   node_modules/stackback/README.md
	new file:   node_modules/stackback/formatstack.js
	new file:   node_modules/stackback/index.js
	new file:   node_modules/stackback/package.json
	new file:   node_modules/stackback/test.js
	new file:   node_modules/std-env/LICENCE
	new file:   node_modules/std-env/README.md
	new file:   node_modules/std-env/dist/index.cjs
	new file:   node_modules/std-env/dist/index.d.cts
	new file:   node_modules/std-env/dist/index.d.mts
	new file:   node_modules/std-env/dist/index.d.ts
	new file:   node_modules/std-env/dist/index.mjs
	new file:   node_modules/std-env/package.json
	new file:   node_modules/strip-literal/LICENSE
	new file:   node_modules/strip-literal/README.md
	new file:   node_modules/strip-literal/dist/index.cjs
	new file:   node_modules/strip-literal/dist/index.d.cts
	new file:   node_modules/strip-literal/dist/index.d.mts
	new file:   node_modules/strip-literal/dist/index.d.ts
	new file:   node_modules/strip-literal/dist/index.mjs
	new file:   node_modules/strip-literal/package.json
	new file:   node_modules/tinybench/LICENSE
	new file:   node_modules/tinybench/README.md
	new file:   node_modules/tinybench/dist/index.cjs
	new file:   node_modules/tinybench/dist/index.d.cts
	new file:   node_modules/tinybench/dist/index.d.ts
	new file:   node_modules/tinybench/dist/index.js
	new file:   node_modules/tinybench/package.json
	new file:   node_modules/tinyexec/LICENSE
	new file:   node_modules/tinyexec/README.md
	new file:   node_modules/tinyexec/dist/main.cjs
	new file:   node_modules/tinyexec/dist/main.d.cts
	new file:   node_modules/tinyexec/dist/main.d.ts
	new file:   node_modules/tinyexec/dist/main.js
	new file:   node_modules/tinyexec/package.json
	new file:   node_modules/tinyglobby/LICENSE
	new file:   node_modules/tinyglobby/README.md
	new file:   node_modules/tinyglobby/dist/index.cjs
	new file:   node_modules/tinyglobby/dist/index.d.cts
	new file:   node_modules/tinyglobby/dist/index.d.mts
	new file:   node_modules/tinyglobby/dist/index.mjs
	new file:   node_modules/tinyglobby/package.json
	new file:   node_modules/tinypool/LICENSE
	new file:   node_modules/tinypool/README.md
	new file:   node_modules/tinypool/dist/common-Qw-RoVFD.js
	new file:   node_modules/tinypool/dist/entry/process.d.ts
	new file:   node_modules/tinypool/dist/entry/process.js
	new file:   node_modules/tinypool/dist/entry/utils.d.ts
	new file:   node_modules/tinypool/dist/entry/utils.js
	new file:   node_modules/tinypool/dist/entry/worker.d.ts
	new file:   node_modules/tinypool/dist/entry/worker.js
	new file:   node_modules/tinypool/dist/index.d.ts
	new file:   node_modules/tinypool/dist/index.js
	new file:   node_modules/tinypool/dist/utils-B--2TaWv.js
	new file:   node_modules/tinypool/dist/utils-De75vAgL.js
	new file:   node_modules/tinypool/package.json
	new file:   node_modules/tinyrainbow/LICENCE
	new file:   node_modules/tinyrainbow/README.md
	new file:   node_modules/tinyrainbow/dist/browser.d.ts
	new file:   node_modules/tinyrainbow/dist/browser.js
	new file:   node_modules/tinyrainbow/dist/chunk-BVHSVHOK.js
	new file:   node_modules/tinyrainbow/dist/index-8b61d5bc.d.ts
	new file:   node_modules/tinyrainbow/dist/node.d.ts
	new file:   node_modules/tinyrainbow/dist/node.js
	new file:   node_modules/tinyrainbow/package.json
	new file:   node_modules/tinyspy/LICENCE
	new file:   node_modules/tinyspy/README.md
	new file:   node_modules/tinyspy/dist/index.d.ts
	new file:   node_modules/tinyspy/dist/index.js
	new file:   node_modules/tinyspy/package.json
	new file:   node_modules/typescript/LICENSE.txt
	new file:   node_modules/typescript/README.md
	new file:   node_modules/typescript/SECURITY.md
	new file:   node_modules/typescript/ThirdPartyNoticeText.txt
	new file:   node_modules/typescript/bin/tsc
	new file:   node_modules/typescript/bin/tsserver
	new file:   node_modules/typescript/lib/_tsc.js
	new file:   node_modules/typescript/lib/_tsserver.js
	new file:   node_modules/typescript/lib/_typingsInstaller.js
	new file:   node_modules/typescript/lib/cs/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/de/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/es/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/fr/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/it/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/ja/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/ko/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/lib.d.ts
	new file:   node_modules/typescript/lib/lib.decorators.d.ts
	new file:   node_modules/typescript/lib/lib.decorators.legacy.d.ts
	new file:   node_modules/typescript/lib/lib.dom.asynciterable.d.ts
	new file:   node_modules/typescript/lib/lib.dom.d.ts
	new file:   node_modules/typescript/lib/lib.dom.iterable.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.collection.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.core.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.generator.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.iterable.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.promise.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.proxy.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.reflect.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.symbol.d.ts
	new file:   node_modules/typescript/lib/lib.es2015.symbol.wellknown.d.ts
	new file:   node_modules/typescript/lib/lib.es2016.array.include.d.ts
	new file:   node_modules/typescript/lib/lib.es2016.d.ts
	new file:   node_modules/typescript/lib/lib.es2016.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2016.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.arraybuffer.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.date.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.object.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.sharedmemory.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.string.d.ts
	new file:   node_modules/typescript/lib/lib.es2017.typedarrays.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.asyncgenerator.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.asynciterable.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.promise.d.ts
	new file:   node_modules/typescript/lib/lib.es2018.regexp.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.array.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.object.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.string.d.ts
	new file:   node_modules/typescript/lib/lib.es2019.symbol.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.bigint.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.date.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.number.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.promise.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.sharedmemory.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.string.d.ts
	new file:   node_modules/typescript/lib/lib.es2020.symbol.wellknown.d.ts
	new file:   node_modules/typescript/lib/lib.es2021.d.ts
	new file:   node_modules/typescript/lib/lib.es2021.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2021.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2021.promise.d.ts
	new file:   node_modules/typescript/lib/lib.es2021.string.d.ts
	new file:   node_modules/typescript/lib/lib.es2021.weakref.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.array.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.error.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.object.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.regexp.d.ts
	new file:   node_modules/typescript/lib/lib.es2022.string.d.ts
	new file:   node_modules/typescript/lib/lib.es2023.array.d.ts
	new file:   node_modules/typescript/lib/lib.es2023.collection.d.ts
	new file:   node_modules/typescript/lib/lib.es2023.d.ts
	new file:   node_modules/typescript/lib/lib.es2023.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2023.intl.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.arraybuffer.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.collection.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.full.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.object.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.promise.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.regexp.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.sharedmemory.d.ts
	new file:   node_modules/typescript/lib/lib.es2024.string.d.ts
	new file:   node_modules/typescript/lib/lib.es5.d.ts
	new file:   node_modules/typescript/lib/lib.es6.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.array.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.collection.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.decorators.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.disposable.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.error.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.float16.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.full.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.intl.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.iterator.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.promise.d.ts
	new file:   node_modules/typescript/lib/lib.esnext.sharedmemory.d.ts
	new file:   node_modules/typescript/lib/lib.scripthost.d.ts
	new file:   node_modules/typescript/lib/lib.webworker.asynciterable.d.ts
	new file:   node_modules/typescript/lib/lib.webworker.d.ts
	new file:   node_modules/typescript/lib/lib.webworker.importscripts.d.ts
	new file:   node_modules/typescript/lib/lib.webworker.iterable.d.ts
	new file:   node_modules/typescript/lib/pl/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/pt-br/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/ru/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/tr/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/tsc.js
	new file:   node_modules/typescript/lib/tsserver.js
	new file:   node_modules/typescript/lib/tsserverlibrary.d.ts
	new file:   node_modules/typescript/lib/tsserverlibrary.js
	new file:   node_modules/typescript/lib/typesMap.json
	new file:   node_modules/typescript/lib/typescript.d.ts
	new file:   node_modules/typescript/lib/typescript.js
	new file:   node_modules/typescript/lib/typingsInstaller.js
	new file:   node_modules/typescript/lib/watchGuard.js
	new file:   node_modules/typescript/lib/zh-cn/diagnosticMessages.generated.json
	new file:   node_modules/typescript/lib/zh-tw/diagnosticMessages.generated.json
	new file:   node_modules/typescript/package.json
	new file:   node_modules/vite-node/LICENSE
	new file:   node_modules/vite-node/README.md
	new file:   node_modules/vite-node/dist/chunk-browser.cjs
	new file:   node_modules/vite-node/dist/chunk-browser.mjs
	new file:   node_modules/vite-node/dist/chunk-hmr.cjs
	new file:   node_modules/vite-node/dist/chunk-hmr.mjs
	new file:   node_modules/vite-node/dist/cli.cjs
	new file:   node_modules/vite-node/dist/cli.d.ts
	new file:   node_modules/vite-node/dist/cli.mjs
	new file:   node_modules/vite-node/dist/client.cjs
	new file:   node_modules/vite-node/dist/client.d.ts
	new file:   node_modules/vite-node/dist/client.mjs
	new file:   node_modules/vite-node/dist/constants.cjs
	new file:   node_modules/vite-node/dist/constants.d.ts
	new file:   node_modules/vite-node/dist/constants.mjs
	new file:   node_modules/vite-node/dist/hmr.cjs
	new file:   node_modules/vite-node/dist/hmr.d.ts
	new file:   node_modules/vite-node/dist/hmr.mjs
	new file:   node_modules/vite-node/dist/index.cjs
	new file:   node_modules/vite-node/dist/index.d-DGmxD2U7.d.ts
	new file:   node_modules/vite-node/dist/index.d.ts
	new file:   node_modules/vite-node/dist/index.mjs
	new file:   node_modules/vite-node/dist/server.cjs
	new file:   node_modules/vite-node/dist/server.d.ts
	new file:   node_modules/vite-node/dist/server.mjs
	new file:   node_modules/vite-node/dist/source-map.cjs
	new file:   node_modules/vite-node/dist/source-map.d.ts
	new file:   node_modules/vite-node/dist/source-map.mjs
	new file:   node_modules/vite-node/dist/trace-mapping.d-DLVdEqOp.d.ts
	new file:   node_modules/vite-node/dist/types.cjs
	new file:   node_modules/vite-node/dist/types.d.ts
	new file:   node_modules/vite-node/dist/types.mjs
	new file:   node_modules/vite-node/dist/utils.cjs
	new file:   node_modules/vite-node/dist/utils.d.ts
	new file:   node_modules/vite-node/dist/utils.mjs
	new file:   node_modules/vite-node/package.json
	new file:   node_modules/vite-node/vite-node.mjs
	new file:   node_modules/vite/LICENSE.md
	new file:   node_modules/vite/README.md
	new file:   node_modules/vite/bin/openChrome.js
	new file:   node_modules/vite/bin/vite.js
	new file:   node_modules/vite/client.d.ts
	new file:   node_modules/vite/dist/client/client.mjs
	new file:   node_modules/vite/dist/client/env.mjs
	new file:   node_modules/vite/dist/node/chunks/build.js
	new file:   node_modules/vite/dist/node/chunks/build2.js
	new file:   node_modules/vite/dist/node/chunks/chunk.js
	new file:   node_modules/vite/dist/node/chunks/config.js
	new file:   node_modules/vite/dist/node/chunks/config2.js
	new file:   node_modules/vite/dist/node/chunks/dist.js
	new file:   node_modules/vite/dist/node/chunks/lib.js
	new file:   node_modules/vite/dist/node/chunks/logger.js
	new file:   node_modules/vite/dist/node/chunks/moduleRunnerTransport.d.ts
	new file:   node_modules/vite/dist/node/chunks/optimizer.js
	new file:   node_modules/vite/dist/node/chunks/postcss-import.js
	new file:   node_modules/vite/dist/node/chunks/preview.js
	new file:   node_modules/vite/dist/node/chunks/server.js
	new file:   node_modules/vite/dist/node/cli.js
	new file:   node_modules/vite/dist/node/index.d.ts
	new file:   node_modules/vite/dist/node/index.js
	new file:   node_modules/vite/dist/node/module-runner.d.ts
	new file:   node_modules/vite/dist/node/module-runner.js
	new file:   node_modules/vite/misc/false.js
	new file:   node_modules/vite/misc/true.js
	new file:   node_modules/vite/package.json
	new file:   node_modules/vite/types/customEvent.d.ts
	new file:   node_modules/vite/types/hmrPayload.d.ts
	new file:   node_modules/vite/types/hot.d.ts
	new file:   node_modules/vite/types/import-meta.d.ts
	new file:   node_modules/vite/types/importGlob.d.ts
	new file:   node_modules/vite/types/importMeta.d.ts
	new file:   node_modules/vite/types/internal/cssPreprocessorOptions.d.ts
	new file:   node_modules/vite/types/internal/lightningcssOptions.d.ts
	new file:   node_modules/vite/types/internal/terserOptions.d.ts
	new file:   node_modules/vite/types/metadata.d.ts
	new file:   node_modules/vite/types/package.json
	new file:   node_modules/vitest/LICENSE.md
	new file:   node_modules/vitest/README.md
	new file:   node_modules/vitest/browser.d.ts
	new file:   node_modules/vitest/config.d.ts
	new file:   node_modules/vitest/coverage.d.ts
	new file:   node_modules/vitest/dist/browser.d.ts
	new file:   node_modules/vitest/dist/browser.js
	new file:   node_modules/vitest/dist/chunks/_commonjsHelpers.BFTU3MAI.js
	new file:   node_modules/vitest/dist/chunks/base.DfmxU-tU.js
	new file:   node_modules/vitest/dist/chunks/benchmark.CYdenmiT.js
	new file:   node_modules/vitest/dist/chunks/benchmark.d.BwvBVTda.d.ts
	new file:   node_modules/vitest/dist/chunks/cac.Cb-PYCCB.js
	new file:   node_modules/vitest/dist/chunks/cli-api.BkDphVBG.js
	new file:   node_modules/vitest/dist/chunks/config.d.D2ROskhv.d.ts
	new file:   node_modules/vitest/dist/chunks/console.CtFJOzRO.js
	new file:   node_modules/vitest/dist/chunks/constants.DnKduX2e.js
	new file:   node_modules/vitest/dist/chunks/coverage.DL5VHqXY.js
	new file:   node_modules/vitest/dist/chunks/coverage.DVF1vEu8.js
	new file:   node_modules/vitest/dist/chunks/coverage.d.S9RMNXIe.d.ts
	new file:   node_modules/vitest/dist/chunks/creator.GK6I-cL4.js
	new file:   node_modules/vitest/dist/chunks/date.Bq6ZW5rf.js
	new file:   node_modules/vitest/dist/chunks/defaults.B7q_naMc.js
	new file:   node_modules/vitest/dist/chunks/env.D4Lgay0q.js
	new file:   node_modules/vitest/dist/chunks/environment.d.cL3nLXbE.d.ts
	new file:   node_modules/vitest/dist/chunks/execute.B7h3T_Hc.js
	new file:   node_modules/vitest/dist/chunks/git.BVQ8w_Sw.js
	new file:   node_modules/vitest/dist/chunks/global.d.MAmajcmJ.d.ts
	new file:   node_modules/vitest/dist/chunks/globals.DEHgCU4V.js
	new file:   node_modules/vitest/dist/chunks/index.B521nVV-.js
	new file:   node_modules/vitest/dist/chunks/index.BCWujgDG.js
	new file:   node_modules/vitest/dist/chunks/index.CdQS2e2Q.js
	new file:   node_modules/vitest/dist/chunks/index.CmSc2RE5.js
	new file:   node_modules/vitest/dist/chunks/index.CwejwG0H.js
	new file:   node_modules/vitest/dist/chunks/index.D3XRDfWc.js
	new file:   node_modules/vitest/dist/chunks/index.VByaPkjc.js
	new file:   node_modules/vitest/dist/chunks/index.X0nbfr6-.js
	new file:   node_modules/vitest/dist/chunks/inspector.C914Efll.js
	new file:   node_modules/vitest/dist/chunks/mocker.d.BE_2ls6u.d.ts
	new file:   node_modules/vitest/dist/chunks/node.fjCdwEIl.js
	new file:   node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts
	new file:   node_modules/vitest/dist/chunks/rpc.-pEldfrD.js
	new file:   node_modules/vitest/dist/chunks/runBaseTests.9Ij9_de-.js
	new file:   node_modules/vitest/dist/chunks/setup-common.Dd054P77.js
	new file:   node_modules/vitest/dist/chunks/suite.d.FvehnV49.d.ts
	new file:   node_modules/vitest/dist/chunks/typechecker.DRKU1-1g.js
	new file:   node_modules/vitest/dist/chunks/utils.CAioKnHs.js
	new file:   node_modules/vitest/dist/chunks/utils.XdZDrNZV.js
	new file:   node_modules/vitest/dist/chunks/vi.bdSIJ99Y.js
	new file:   node_modules/vitest/dist/chunks/vite.d.CMLlLIFP.d.ts
	new file:   node_modules/vitest/dist/chunks/vm.BThCzidc.js
	new file:   node_modules/vitest/dist/chunks/worker.d.1GmBbd7G.d.ts
	new file:   node_modules/vitest/dist/chunks/worker.d.CKwWzBSj.d.ts
	new file:   node_modules/vitest/dist/cli.js
	new file:   node_modules/vitest/dist/config.cjs
	new file:   node_modules/vitest/dist/config.d.ts
	new file:   node_modules/vitest/dist/config.js
	new file:   node_modules/vitest/dist/coverage.d.ts
	new file:   node_modules/vitest/dist/coverage.js
	new file:   node_modules/vitest/dist/environments.d.ts
	new file:   node_modules/vitest/dist/environments.js
	new file:   node_modules/vitest/dist/execute.d.ts
	new file:   node_modules/vitest/dist/execute.js
	new file:   node_modules/vitest/dist/index.d.ts
	new file:   node_modules/vitest/dist/index.js
	new file:   node_modules/vitest/dist/mocker.d.ts
	new file:   node_modules/vitest/dist/mocker.js
	new file:   node_modules/vitest/dist/node.d.ts
	new file:   node_modules/vitest/dist/node.js
	new file:   node_modules/vitest/dist/path.js
	new file:   node_modules/vitest/dist/reporters.d.ts
	new file:   node_modules/vitest/dist/reporters.js
	new file:   node_modules/vitest/dist/runners.d.ts
	new file:   node_modules/vitest/dist/runners.js
	new file:   node_modules/vitest/dist/snapshot.d.ts
	new file:   node_modules/vitest/dist/snapshot.js
	new file:   node_modules/vitest/dist/spy.js
	new file:   node_modules/vitest/dist/suite.d.ts
	new file:   node_modules/vitest/dist/suite.js
	new file:   node_modules/vitest/dist/worker.js
	new file:   node_modules/vitest/dist/workers.d.ts
	new file:   node_modules/vitest/dist/workers.js
	new file:   node_modules/vitest/dist/workers/forks.js
	new file:   node_modules/vitest/dist/workers/runVmTests.js
	new file:   node_modules/vitest/dist/workers/threads.js
	new file:   node_modules/vitest/dist/workers/vmForks.js
	new file:   node_modules/vitest/dist/workers/vmThreads.js
	new file:   node_modules/vitest/environments.d.ts
	new file:   node_modules/vitest/execute.d.ts
	new file:   node_modules/vitest/globals.d.ts
	new file:   node_modules/vitest/import-meta.d.ts
	new file:   node_modules/vitest/importMeta.d.ts
	new file:   node_modules/vitest/index.cjs
	new file:   node_modules/vitest/index.d.cts
	new file:   node_modules/vitest/jsdom.d.ts
	new file:   node_modules/vitest/mocker.d.ts
	new file:   node_modules/vitest/node.d.ts
	new file:   node_modules/vitest/optional-types.d.ts
	new file:   node_modules/vitest/package.json
	new file:   node_modules/vitest/reporters.d.ts
	new file:   node_modules/vitest/runners.d.ts
	new file:   node_modules/vitest/snapshot.d.ts
	new file:   node_modules/vitest/suite.d.ts
	new file:   node_modules/vitest/suppress-warnings.cjs
	new file:   node_modules/vitest/utils.d.ts
	new file:   node_modules/vitest/vitest.mjs
	new file:   node_modules/vitest/workers.d.ts
	new file:   node_modules/why-is-node-running/.github/FUNDING.yml
	new file:   node_modules/why-is-node-running/LICENSE
	new file:   node_modules/why-is-node-running/README.md
	new file:   node_modules/why-is-node-running/cli.js
	new file:   node_modules/why-is-node-running/example.js
	new file:   node_modules/why-is-node-running/include.js
	new file:   node_modules/why-is-node-running/index.js
	new file:   node_modules/why-is-node-running/package.json
	new file:   prompt.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   prompt.md



[Showing lines 9-940 of 940 (50.0KB limit). Full output: /tmp/pi-bash-a8bf0d1dc5bc8002.log]

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
ls: cannot access 'reports': No such file or directory


Command exited with code 2

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
setup.mjs


@[assistant]:


@[toolResult]:
#!/usr/bin/env node
/**
 * Workspace before_all hook: copy autopilot-dev skills into the workspace
 * for agent discovery. Receives workspace_path via stdin JSON from AgentV.
 */

import { execSync } from 'node:child_process';
import { cpSync, mkdirSync, readFileSync, readdirSync } from 'node:fs';
import { join } from 'node:path';

// Read workspace_path from stdin (provided by AgentV orchestrator)
let workspacePath;
try {
  const stdin = readFileSync(0, 'utf8');
  const context = JSON.parse(stdin);
  workspacePath = context.workspace_path;
} catch {
  workspacePath = process.cwd();
}

// Resolve repo root from cwd (eval dir is inside the repo)
let repoRoot;
try {
  repoRoot = execSync('git rev-parse --show-toplevel', {
    encoding: 'utf8',
  }).trim();
} catch {
  console.error('Failed to resolve repo root from cwd:', process.cwd());
  process.exit(1);
}

console.log(`Workspace: ${workspacePath}`);
console.log(`Repo root: ${repoRoot}`);

// Copy to skill discovery directories in the workspace
// Each provider discovers skills from a different path:
//   Claude CLI: .claude/skills/
//   Pi CLI / Pi Coding Agent: .agents/skills/
//   Codex: .agents/skills/ or .codex/skills/
const skillDirs = [
  join(workspacePath, '.claude', 'skills'),
  join(workspacePath, '.agents', 'skills'),
  join(workspacePath, '.pi', 'skills'),
];
for (const dir of skillDirs) {
  mkdirSync(dir, { recursive: true });
}

// Copy all autopilot-dev skills
const pluginSkillsDir = join(repoRoot, 'plugins', 'autopilot-dev', 'skills');
const skillNames = readdirSync(pluginSkillsDir);

for (const name of skillNames) {
  const src = join(pluginSkillsDir, name);
  for (const dir of skillDirs) {
    cpSync(src, join(dir, name), { recursive: true });
  }
  console.log(`Copied ${name}`);
}

for (const dir of skillDirs) {
  console.log(`Skills in ${dir}: ${readdirSync(dir).join(', ')}`);
}

// Initialize git repo in workspace so ship/claim tests can use git commands
try {
  execSync('git init && git add -A && git commit -m "initial commit"', {
    cwd: workspacePath,
    encoding: 'utf8',
    stdio: 'pipe',
  });
  execSync('git checkout -b feat/42-add-priority', {
    cwd: workspacePath,
    encoding: 'utf8',
    stdio: 'pipe',
  });
  console.log('Git repo initialized with feat branch');
} catch (e) {
  console.error('Git init failed:', e.message);
}


@[assistant]:


@[toolResult]:
@[User]:
<file: path="plugins/autopilot-dev/skills/ad-verify/SKILL.md">

@[User]:
Verify the priority field works. The AGENTS.md says both CLI and API modes
use the same service layer and both must be tested.


@[assistant]:


@[toolResult]:
(no output)

@[assistant]:


@[toolResult]:
task-service.ts


@[assistant]:


@[toolResult]:
.
./.claude
./.claude/skills
./.claude/skills/ad-implement
./.claude/skills/ad-ship
./.claude/skills/ad-using-autopilot-dev
./.claude/skills/ad-design
./.claude/skills/ad-plan
./.claude/skills/ad-explore
./.claude/skills/ad-claim
./.claude/skills/ad-verify
./.git
./.git/info
./.git/refs
./.git/refs/heads
./.git/refs/tags
./.git/objects
./.git/objects/info
./.git/objects/c3
./.git/objects/ea
./.git/objects/55
./.git/objects/11
./.git/objects/0d
./.git/objects/98
./.git/objects/21
./.git/objects/7e
./.git/objects/8d
./.git/objects/a3
./.git/objects/df
./.git/objects/63
./.git/objects/f9
./.git/objects/8c
./.git/objects/62
./.git/objects/42
./.git/objects/de
./.git/objects/9a
./.git/objects/04
./.git/objects/4c
./.git/objects/20
./.git/objects/69
./.git/objects/pack
./.git/objects/b1
./.git/objects/71
./.git/objects/19
./.git/objects/a7
./.git/objects/be
./.git/objects/82
./.git/objects/a0
./.git/objects/3e
./.git/objects/d8
./.git/objects/c2
./.git/objects/88
./.git/objects/91
./.git/objects/66
./.git/objects/ef
./.git/objects/c9
./.git/objects/59
./.git/objects/99
./.git/objects/92
./.git/objects/0b
./.git/objects/9f
./.git/objects/da
./.git/objects/fb
./.git/objects/34
./.git/objects/6c
./.git/objects/c1
./.git/objects/80
./.git/objects/b8
./.git/objects/05
./.git/objects/d2
./.git/objects/c8
./.git/objects/1d
./.git/objects/70
./.git/objects/e1
./.git/objects/83
./.git/objects/64
./.git/objects/49
./.git/objects/c7
./.git/objects/4d
./.git/objects/0c
./.git/objects/41
./.git/objects/86
./.git/objects/e6
./.git/objects/f7
./.git/objects/79
./.git/objects/31
./.git/objects/74
./.git/objects/1b
./.git/objects/f1
./.git/objects/01
./.git/objects/9d
./.git/objects/9c
./.git/objects/ee
./.git/objects/3d
./.git/objects/af
./.git/objects/f3
./.git/objects/ab
./.git/objects/ba
./.git/objects/0f
./.git/objects/7b
./.git/objects/7d
./.git/objects/54
./.git/objects/b2
./.git/objects/23
./.git/objects/f0
./.git/objects/84
./.git/objects/67
./.git/objects/b6
./.git/objects/d7
./.git/objects/fd
./.git/objects/50
./.git/objects/a9
./.git/objects/12
./.git/objects/51
./.git/objects/37
./.git/objects/1a
./.git/objects/a8
./.git/objects/87
./.git/objects/6f
./.git/objects/e3
./.git/objects/5d
./.git/objects/6a
./.git/objects/cb
./.git/objects/38
./.git/objects/f8
./.git/objects/57
./.git/objects/26
./.git/objects/78
./.git/objects/5f
./.git/objects/c6
./.git/objects/2c
./.git/objects/0a
./.git/objects/40
./.git/objects/65
./.git/objects/32
./.git/objects/2d
./.git/objects/e2
./.git/objects/53
./.git/objects/0e
./.git/objects/90
./.git/objects/1e
./.git/objects/ed
./.git/objects/ae
./.git/objects/8a
./.git/objects/2b
./.git/objects/73
./.git/objects/3f
./.git/objects/16
./.git/objects/03
./.git/objects/02
./.git/objects/61
./.git/objects/fa
./.git/objects/46
./.git/objects/bd
./.git/objects/6e
./.git/objects/c0
./.git/objects/9b
./.git/objects/7a
./.git/objects/33
./.git/objects/e5
./.git/objects/43
./.git/objects/d9
./.git/objects/94
./.git/objects/48
./.git/objects/2e
./.git/objects/2a
./.git/objects/7c
./.git/objects/39
./.git/objects/aa
./.git/objects/d3
./.git/objects/9e
./.git/objects/8b
./.git/objects/a6
./.git/objects/2f
./.git/objects/f4
./.git/objects/52
./.git/objects/bb
./.git/objects/56
./.git/objects/3a
./.git/objects/45
./.git/objects/4a
./.git/objects/27
./.git/objects/a5
./.git/objects/17
./.git/objects/58
./.git/objects/e7
./.git/objects/7f
./.git/objects/5b
./.git/objects/25
./.git/objects/e8
./.git/objects/a2
./.git/objects/b5
./.git/objects/85
./.git/objects/95
./.git/objects/18
./.git/objects/93
./.git/objects/00
./.git/objects/d4
./.git/objects/44
./.git/objects/f6
./.git/objects/a1
./.git/objects/ff
./.git/objects/07
./.git/objects/b7
./.git/objects/06
./.git/objects/ca
./.git/objects/5c
./.git/objects/22
./.git/objects/ce
./.git/objects/96
./.git/objects/97
./.git/objects/77
./.git/objects/6b
./.git/objects/72
./.git/objects/bf
./.git/objects/f2
./.git/objects/15
./.git/objects/6d
./.git/objects/b3
./.git/objects/dc
./.git/objects/60
./.git/objects/f5
./.git/objects/3c
./.git/objects/fe
./.git/objects/5e
./.git/objects/08
./.git/objects/dd
./.git/objects/76
./.git/objects/ec
./.git/objects/ad
./.git/objects/1f
./.git/objects/75
./.git/objects/28
./.git/objects/5a
./.git/objects/10
./.git/objects/cc
./.git/objects/81
./.git/objects/d5
./.git/objects/13
./.git/objects/35
./.git/objects/db
./.git/objects/cf
./.git/objects/cd
./.git/objects/09
./.git/objects/8f
./.git/objects/4f
./.git/objects/36
./.git/objects/d6
./.git/objects/e9
./.git/objects/3b
./.git/objects/a4
./.git/objects/b4
./.git/objects/d1
./.git/objects/14
./.git/objects/b0
./.git/objects/c5
./.git/objects/68
./.git/objects/eb
./.git/objects/d0
./.git/objects/bc
./.git/objects/e0
./.git/objects/e4
./.git/objects/8e
./.git/objects/4b
./.git/objects/89
./.git/hooks
./.git/branches
./.git/logs
./.git/logs/refs
./.pi
./.pi/skills
./.pi/skills/ad-implement
./.pi/skills/ad-ship
./.pi/skills/ad-using-autopilot-dev
./.pi/skills/ad-design
./.pi/skills/ad-plan
./.pi/skills/ad-explore
./.pi/skills/ad-claim
./.pi/skills/ad-verify
./src
./src/cli
./src/utils
./src/reports
./src/services
./src/api
./src/models
./node_modules
./node_modules/postcss
./node_modules/postcss/lib
./node_modules/es-module-lexer
./node_modules/es-module-lexer/dist
./node_modules/es-module-lexer/types
./node_modules/picocolors
./node_modules/tinyexec
./node_modules/tinyexec/dist
./node_modules/stackback
./node_modules/fdir
./node_modules/fdir/dist
./node_modules/siginfo
./node_modules/@rollup
./node_modules/@rollup/rollup-linux-arm64-musl
./node_modules/@rollup/rollup-linux-arm64-gnu
./node_modules/vite-node
./node_modules/vite-node/dist
./node_modules/nanoid
./node_modules/nanoid/async
./node_modules/nanoid/bin
./node_modules/nanoid/non-secure
./node_modules/nanoid/url-alphabet
./node_modules/vitest
./node_modules/vitest/dist
./node_modules/chai
./node_modules/chai/lib
./node_modules/tinyspy
./node_modules/tinyspy/dist
./node_modules/check-error
./node_modules/expect-type
./node_modules/expect-type/dist
./node_modules/@esbuild
./node_modules/@esbuild/linux-arm64
./node_modules/@biomejs
./node_modules/@biomejs/cli-linux-arm64-musl
./node_modules/@biomejs/cli-linux-arm64
./node_modules/@biomejs/biome
./node_modules/std-env
./node_modules/std-env/dist
./node_modules/deep-eql
./node_modules/pathval
./node_modules/vite
./node_modules/vite/bin
./node_modules/vite/misc
./node_modules/vite/dist
./node_modules/vite/types
./node_modules/@jridgewell
./node_modules/@jridgewell/sourcemap-codec
./node_modules/why-is-node-running
./node_modules/why-is-node-running/.github
./node_modules/typescript
./node_modules/typescript/bin
./node_modules/typescript/lib
./node_modules/rollup
./node_modules/rollup/dist
./node_modules/cac
./node_modules/cac/dist
./node_modules/cac/deno
./node_modules/ms
./node_modules/tinyrainbow
./node_modules/tinyrainbow/dist
./node_modules/magic-string
./node_modules/magic-string/dist
./node_modules/tinyglobby
./node_modules/tinyglobby/dist
./node_modules/loupe
./node_modules/loupe/lib
./node_modules/@vitest
./node_modules/@vitest/spy
./node_modules/@vitest/utils
./node_modules/@vitest/expect
./node_modules/@vitest/snapshot
./node_modules/@vitest/runner
./node_modules/@vitest/mocker
./node_modules/@vitest/pretty-format
./node_modules/@types
./node_modules/@types/chai
./node_modules/@types/deep-eql
./node_modules/@types/estree
./node_modules/source-map-js
./node_modules/source-map-js/lib
./node_modules/.bin
./node_modules/strip-literal
./node_modules/strip-literal/dist
./node_modules/js-tokens
./node_modules/assertion-error
./node_modules/esbuild
./node_modules/esbuild/bin
./node_modules/esbuild/lib
./node_modules/estree-walker
./node_modules/estree-walker/src
./node_modules/estree-walker/types
./node_modules/debug
./node_modules/debug/src
./node_modules/pathe
./node_modules/pathe/dist
./node_modules/tinybench
./node_modules/tinybench/dist
./node_modules/tinypool
./node_modules/tinypool/dist
./node_modules/picomatch
./node_modules/picomatch/lib
./scripts
./.agents
./.agents/skills
./.agents/skills/ad-implement
./.agents/skills/ad-ship
./.agents/skills/ad-using-autopilot-dev
./.agents/skills/ad-design
./.agents/skills/ad-plan
./.agents/skills/ad-explore
./.agents/skills/ad-claim
./.agents/skills/ad-verify


@[assistant]:


@[toolResult]:
$ biome check .
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
  

check ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━Checked 10 files in 12ms. No fixes applied.
Found 7 errors.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  × Some errors were emitted while running checks.
  

error: script "lint" exited with code 1


Command exited with code 1

@[assistant]:


@[toolResult]:
$ biome check . --write
Checked 10 files in 21ms. Fixed 7 files.


@[assistant]:


@[toolResult]:
## feat/42-add-priority
A  bun.lock
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
A  node_modules/@biomejs/cli-linux-arm64-musl/biome
A  node_modules/@biomejs/cli-linux-arm64-musl/package.json
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
A  node_modules/@rollup/rollup-linux-arm64-musl/README.md
A  node_modules/@rollup/rollup-linux-arm64-musl/package.json
A  node_modules/@rollup/rollup-linux-arm64-musl/rollup.linux-arm64-musl.node
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
AM prompt.md
 M scripts/setup.mjs
 M src/api/index.ts
 M src/cli/index.ts
 M src/models/task.ts
 M src/reports/summary.ts
 M src/services/task-service.ts
 M src/utils/format-task.ts


@[assistant]:


@[toolResult]:
diff --git a/scripts/setup.mjs b/scripts/setup.mjs
index 6fbdc5a..e753823 100644
--- a/scripts/setup.mjs
+++ b/scripts/setup.mjs
@@ -4,14 +4,14 @@
  * for agent discovery. Receives workspace_path via stdin JSON from AgentV.
  */
 
-import { execSync } from 'node:child_process';
-import { cpSync, mkdirSync, readFileSync, readdirSync } from 'node:fs';
-import { join } from 'node:path';
+import { execSync } from "node:child_process";
+import { cpSync, mkdirSync, readFileSync, readdirSync } from "node:fs";
+import { join } from "node:path";
 
 // Read workspace_path from stdin (provided by AgentV orchestrator)
 let workspacePath;
 try {
-  const stdin = readFileSync(0, 'utf8');
+  const stdin = readFileSync(0, "utf8");
   const context = JSON.parse(stdin);
   workspacePath = context.workspace_path;
 } catch {
@@ -21,11 +21,11 @@ try {
 // Resolve repo root from cwd (eval dir is inside the repo)
 let repoRoot;
 try {
-  repoRoot = execSync('git rev-parse --show-toplevel', {
-    encoding: 'utf8',
+  repoRoot = execSync("git rev-parse --show-toplevel", {
+    encoding: "utf8",
   }).trim();
 } catch {
-  console.error('Failed to resolve repo root from cwd:', process.cwd());
+  console.error("Failed to resolve repo root from cwd:", process.cwd());
   process.exit(1);
 }
 
@@ -38,16 +38,16 @@ console.log(`Repo root: ${repoRoot}`);
 //   Pi CLI / Pi Coding Agent: .agents/skills/
 //   Codex: .agents/skills/ or .codex/skills/
 const skillDirs = [
-  join(workspacePath, '.claude', 'skills'),
-  join(workspacePath, '.agents', 'skills'),
-  join(workspacePath, '.pi', 'skills'),
+  join(workspacePath, ".claude", "skills"),
+  join(workspacePath, ".agents", "skills"),
+  join(workspacePath, ".pi", "skills"),
 ];
 for (const dir of skillDirs) {
   mkdirSync(dir, { recursive: true });
 }
 
 // Copy all autopilot-dev skills
-const pluginSkillsDir = join(repoRoot, 'plugins', 'autopilot-dev', 'skills');
+const pluginSkillsDir = join(repoRoot, "plugins", "autopilot-dev", "skills");
 const skillNames = readdirSync(pluginSkillsDir);
 
 for (const name of skillNames) {
@@ -59,22 +59,22 @@ for (const name of skillNames) {
 }
 
 for (const dir of skillDirs) {
-  console.log(`Skills in ${dir}: ${readdirSync(dir).join(', ')}`);
+  console.log(`Skills in ${dir}: ${readdirSync(dir).join(", ")}`);
 }
 
 // Initialize git repo in workspace so ship/claim tests can use git commands
 try {
   execSync('git init && git add -A && git commit -m "initial commit"', {
     cwd: workspacePath,
-    encoding: 'utf8',
-    stdio: 'pipe',
+    encoding: "utf8",
+    stdio: "pipe",
   });
-  execSync('git checkout -b feat/42-add-priority', {
+  execSync("git checkout -b feat/42-add-priority", {
     cwd: workspacePath,
-    encoding: 'utf8',
-    stdio: 'pipe',
+    encoding: "utf8",
+    stdio: "pipe",
   });
-  console.log('Git repo initialized with feat branch');
+  console.log("Git repo initialized with feat branch");
 } catch (e) {
-  console.error('Git init failed:', e.message);
+  console.error("Git init failed:", e.message);
 }


@[assistant]:


@[toolResult]:
$ tsc --noEmit


@[assistant]:


@[toolResult]:
$ vitest run

 RUN  v3.2.4 /home/christso/.agentv/workspaces/471b3f14-f6f3-4bb1-bce1-19a7084dbfa1/shared

No test files found, exiting with code 1

include: **/*.{test,spec}.?(c|m)[jt]s?(x)
exclude:  **/node_modules/**, **/dist/**, **/cypress/**, **/.{idea,git,cache,output,temp}/**, **/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build,eslint,prettier}.config.*

error: script "test" exited with code 1


Command exited with code 1

@[assistant]:


@[toolResult]:
ls: cannot access 'plugins': No such file or directory


Command exited with code 2

@[assistant]:


@[toolResult]:
.
..
.agents
AGENTS.md
biome.json
bun.lock
.claude
CLAUDE.md
.git
node_modules
package.json
.pi
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
$ biome check .
Checked 10 files in 22ms. No fixes applied.


@[assistant]:


@[toolResult]:
## feat/42-add-priority
A  bun.lock
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
A  node_modules/@biomejs/cli-linux-arm64-musl/biome
A  node_modules/@biomejs/cli-linux-arm64-musl/package.json
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
A  node_modules/@rollup/rollup-linux-arm64-musl/README.md
A  node_modules/@rollup/rollup-linux-arm64-musl/package.json
A  node_modules/@rollup/rollup-linux-arm64-musl/rollup.linux-arm64-musl.node
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
AM prompt.md
 M scripts/setup.mjs
 M src/api/index.ts
 M src/cli/index.ts
 M src/models/task.ts
 M src/reports/summary.ts
 M src/services/task-service.ts
 M src/utils/format-task.ts


@[assistant]:


@[toolResult]:
src/utils/format-task.ts:14: * Partial implementation of priority derivation.
src/utils/format-task.ts:15: * Currently only handles basic cases — does not support custom priority rules.


@[assistant]:


@[toolResult]:
scripts/setup.mjs:72:  execSync("git checkout -b feat/42-add-priority", {
prompt.md:5:Verify the priority field works. The AGENTS.md says both CLI and API modes
src/utils/format-task.ts:14: * Partial implementation of priority derivation.
src/utils/format-task.ts:15: * Currently only handles basic cases — does not support custom priority rules.
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
node_modules/vitest/dist/environments.d.ts:15:	// has a priority for getting implementation from symbols
node_modules/vitest/dist/chunks/coverage.DL5VHqXY.js:3866: // CLI flag has higher priority
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
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1735:	* Maximum number or percentage of workers to run tests in. `poolOptions.{threads,vmThreads}.maxThreads`/`poolOptions.forks.maxForks` has higher priority.
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1739:	* Minimum number or percentage of workers to run tests in. `poolOptions.{threads,vmThreads}.minThreads`/`poolOptions.forks.minForks` has higher priority.
node_modules/typescript/lib/lib.es2020.string.d.ts:40:     * @param locales A locale string or array of locale strings that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used. This parameter must conform to BCP 47 standards; see the Intl.Collator object for details.
node_modules/typescript/lib/lib.es2020.number.d.ts:24:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es2020.date.d.ts:24:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es2020.date.d.ts:31:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.es2020.date.d.ts:38:     * @param locales A locale string, array of locale strings, Intl.Locale object, or array of Intl.Locale objects that contain one or more language or locale tags. If you include more than one locale string, list them in descending order of priority so that the first entry is the preferred locale. If you omit this parameter, the default locale of the JavaScript runtime is used.
node_modules/typescript/lib/lib.webworker.d.ts:757:    priority?: RequestPriority;
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
Summary:
- Ran the mandated build/test/lint chain from package.json.
  - `bun run build` → ✅ (TypeScript compile passed)
  - `bun run test` → ❌ Vitest exits with code 1 because the repository contains **zero** test files (`No test files found, exiting with code 1`). This prevents completing verification.
  - `bun run lint` → ✅ (Biome check passes after auto-formatting earlier; rerun produced no errors)

Findings:
1. The `Task` model (`src/models/task.ts`) has no `priority` field, and the service layer/CLI/API never read or return a priority. The only priority-related code is `derivePriority()` in `src/utils/format-task.ts`, but its result is unused anywhere in the codebase.
2. Because neither CLI nor API exposes a priority attribute, there is no user-facing behavior to exercise for the required CLI/API E2E checks. Any attempt to “verify the priority field” currently fails by inspection: the feature simply isn’t implemented.
3. Automated verification cannot proceed until actual tests exist. Vitest’s failure (see output above) is a hard gate per `ad-verify`’s Iron Law.

Next steps required to achieve verification:
- Implement the priority field across the shared service layer, CLI output, API responses (and any other consumers like `src/reports/summary.ts`), ensuring both entry points surface it.
- Add Vitest coverage (at least one spec each for CLI and API flows) asserting the expected priority behavior so `bun run test` passes with evidence.
- After implementation, rerun the build/test/lint chain and perform the CLI/API E2E red/green checks with captured output, plus a blast-radius grep for the touched shared utilities.

Until these gaps are addressed, I cannot produce the verification evidence you requested.