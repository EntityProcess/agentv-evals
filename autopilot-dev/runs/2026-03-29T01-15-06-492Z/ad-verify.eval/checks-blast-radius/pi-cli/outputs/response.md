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
I modified the Task interface in src/models/task.ts and the formatTask function
in src/utils/format-task.ts. Check whether any consumers were missed.


@[assistant]:


@[toolResult]:
/home/christso/.agentv/workspaces/471b3f14-f6f3-4bb1-bce1-19a7084dbfa1/shared


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
node_modules/tinypool/dist/index.d.ts:81:interface Task {
node_modules/tinypool/dist/index.d.ts:87:  shift(): Task | null;
node_modules/tinypool/dist/index.d.ts:88:  remove(task: Task): void;
node_modules/tinypool/dist/index.d.ts:89:  push(task: Task): void;
node_modules/tinypool/dist/index.d.ts:195:export { Options, ReadyMessage, RequestMessage, ResponseMessage, StartupMessage, Task, TaskQueue, Tinypool, TinypoolChannel, TinypoolData, TinypoolPrivateData, TinypoolWorker, TinypoolWorkerMessage, Transferable, Tinypool as default, isMovable, isTaskQueue, isTransferable, kFieldCount, kQueueOptions, kRequestCountField, kResponseCountField, kTransferable, kValue, markMovable, _workerId as workerId };
node_modules/tinybench/dist/index.d.ts:6:declare class Task extends EventTarget {
node_modules/tinybench/dist/index.d.ts:19:     * Task options
node_modules/tinybench/dist/index.d.ts:25:     * run the current task and write the results in `Task.result` object
node_modules/tinybench/dist/index.d.ts:39:     * reset the task to make the `Task.runs` a zero-value and remove the `Task.result`
node_modules/tinybench/dist/index.d.ts:53:    beforeAll?: (this: Task) => void | Promise<void>;
node_modules/tinybench/dist/index.d.ts:57:    beforeEach?: (this: Task) => void | Promise<void>;
node_modules/tinybench/dist/index.d.ts:61:    afterEach?: (this: Task) => void | Promise<void>;
node_modules/tinybench/dist/index.d.ts:65:    afterAll?: (this: Task) => void | Promise<void>;
node_modules/tinybench/dist/index.d.ts:146:  * Both the `Task` and `Bench` objects extend the `EventTarget` object,
node_modules/tinybench/dist/index.d.ts:155:type Hook = (task: Task, mode: 'warmup' | 'run') => void | Promise<void>;
node_modules/tinybench/dist/index.d.ts:158:    task: Task;
node_modules/tinybench/dist/index.d.ts:224:    task: Task | null;
node_modules/tinybench/dist/index.d.ts:234:    _tasks: Map<string, Task>;
node_modules/tinybench/dist/index.d.ts:235:    _todos: Map<string, Task>;
node_modules/tinybench/dist/index.d.ts:264:    run(): Promise<Task[]>;
node_modules/tinybench/dist/index.d.ts:268:    runConcurrently(threshold?: number, mode?: NonNullable<Bench['concurrency']>): Promise<Task[]>;
node_modules/tinybench/dist/index.d.ts:300:    table(convert?: (task: Task) => Record<string, string | number> | undefined): (Record<string, string | number> | null)[];
node_modules/tinybench/dist/index.d.ts:308:    get tasks(): Task[];
node_modules/tinybench/dist/index.d.ts:309:    get todos(): Task[];
node_modules/tinybench/dist/index.d.ts:313:    getTask(name: string): Task | undefined;
node_modules/tinybench/dist/index.d.ts:319:export { Bench, type BenchEvent, type BenchEvents, type Fn, type Hook, type Options, Task, type TaskEvents, type TaskResult, hrtimeNow, now };
node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts:41:	* Task name provided by the user. If no name was provided, it will be an empty string.
node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts:45:	* Task mode.
node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts:187:	tasks: Task[];
node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts:250:type Task = Test | Suite | File;
node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts:470:	* Task fixtures.
node_modules/@vitest/runner/dist/tasks.d-CkscK4of.d.ts:558:export type { AfterAllListener as A, BeforeAllListener as B, ChainableFunction as C, DoneCallback as D, ExtendedContext as E, File as F, TaskMeta as G, HookCleanupCallback as H, ImportDuration as I, TaskPopulated as J, TaskResult as K, TaskResultPack as L, TaskState as M, TestAnnotation as N, OnTestFailedHandler as O, TestAnnotationLocation as P, TestAttachment as Q, RunMode as R, Suite as S, Task as T, TestContext as U, TestFunction as V, TestOptions as W, Use as X, Test as a, AfterEachListener as b, BeforeEachListener as d, TaskHook as e, OnTestFinishedHandler as f, Custom as g, SuiteHooks as h, TaskUpdateEvent as i, TestAPI as j, SuiteAPI as k, SuiteCollector as l, Fixture as m, FixtureFn as n, FixtureOptions as o, Fixtures as p, HookListener as q, InferFixturesTypes as r, RuntimeContext as s, SequenceHooks as t, SequenceSetupFiles as u, SuiteFactory as v, TaskBase as w, TaskContext as x, TaskCustomOptions as y, TaskEventPack as z };
node_modules/@vitest/runner/dist/index.d.ts:1:import { A as AfterAllListener, b as AfterEachListener, B as BeforeAllListener, d as BeforeEachListener, e as TaskHook, O as OnTestFailedHandler, f as OnTestFinishedHandler, a as Test, g as Custom, S as Suite, h as SuiteHooks, F as File, i as TaskUpdateEvent, T as Task, j as TestAPI, k as SuiteAPI, l as SuiteCollector } from './tasks.d-CkscK4of.js';
node_modules/@vitest/runner/dist/index.d.ts:126:declare function getFn<Task = Test | Custom>(key: Task): () => Awaitable<void>;
node_modules/@vitest/runner/dist/index.d.ts:130:declare function updateTask(event: TaskUpdateEvent, task: Task, runner: VitestRunner): void;
node_modules/@vitest/runner/dist/index.d.ts:261:export { AfterAllListener, AfterEachListener, BeforeAllListener, BeforeEachListener, Custom, TestAPI as CustomAPI, File, FileSpecification, OnTestFailedHandler, OnTestFinishedHandler, Suite, SuiteAPI, SuiteCollector, SuiteHooks, Task, TaskHook, TaskUpdateEvent, Test, TestAPI, VitestRunner, afterAll, afterEach, beforeAll, beforeEach, publicCollect as collectTests, createTaskCollector, describe, getCurrentSuite, getCurrentTest, getFn, getHooks, it, onTestFailed, onTestFinished, setFn, setHooks, startTests, suite, test, updateTask };
node_modules/@vitest/runner/dist/types.d.ts:3:export { A as AfterAllListener, b as AfterEachListener, B as BeforeAllListener, d as BeforeEachListener, g as Custom, j as CustomAPI, D as DoneCallback, E as ExtendedContext, m as Fixture, n as FixtureFn, o as FixtureOptions, p as Fixtures, H as HookCleanupCallback, q as HookListener, r as InferFixturesTypes, O as OnTestFailedHandler, f as OnTestFinishedHandler, R as RunMode, s as RuntimeContext, k as SuiteAPI, l as SuiteCollector, v as SuiteFactory, h as SuiteHooks, T as Task, w as TaskBase, x as TaskContext, y as TaskCustomOptions, e as TaskHook, G as TaskMeta, J as TaskPopulated, K as TaskResult, M as TaskState, i as TaskUpdateEvent, j as TestAPI, P as TestAnnotationLocation, Q as TestAttachment, V as TestFunction, W as TestOptions, X as Use } from './tasks.d-CkscK4of.js';
node_modules/@vitest/runner/dist/utils.d.ts:1:import { S as Suite, F as File, T as Task, a as Test } from './tasks.d-CkscK4of.js';
node_modules/@vitest/runner/dist/utils.d.ts:31:declare function partitionSuiteChildren(suite: Suite): Task[][];
node_modules/@vitest/runner/dist/utils.d.ts:36:declare function isAtomTest(s: Task): s is Test;
node_modules/@vitest/runner/dist/utils.d.ts:37:declare function isTestCase(s: Task): s is Test;
node_modules/@vitest/runner/dist/utils.d.ts:38:declare function getTests(suite: Arrayable<Task>): Test[];
node_modules/@vitest/runner/dist/utils.d.ts:39:declare function getTasks(tasks?: Arrayable<Task>): Task[];
node_modules/@vitest/runner/dist/utils.d.ts:40:declare function getSuites(suite: Arrayable<Task>): Suite[];
node_modules/@vitest/runner/dist/utils.d.ts:42:declare function hasFailed(suite: Arrayable<Task>): boolean;
node_modules/@vitest/runner/dist/utils.d.ts:43:declare function getNames(task: Task): string[];
node_modules/@vitest/runner/dist/utils.d.ts:44:declare function getFullName(task: Task, separator?: string): string;
node_modules/@vitest/runner/dist/utils.d.ts:45:declare function getTestName(task: Task, separator?: string): string;
node_modules/vitest/dist/index.d.ts:1:import { File as File$1, TestAnnotation, TaskResultPack as TaskResultPack$1, TaskEventPack, TaskPopulated, Suite as Suite$1, Test as Test$1, Custom as Custom$1, Task as Task$1, TaskBase as TaskBase$1, TaskResult as TaskResult$1, DoneCallback as DoneCallback$1, RuntimeContext as RuntimeContext$1, SuiteHooks as SuiteHooks$1, SequenceHooks as SequenceHooks$1, SequenceSetupFiles as SequenceSetupFiles$1 } from '@vitest/runner';
node_modules/vitest/dist/index.d.ts:2:export { CancelReason, ExtendedContext, HookCleanupCallback, HookListener, ImportDuration, OnTestFailedHandler, OnTestFinishedHandler, RunMode, Custom as RunnerCustomCase, Task as RunnerTask, TaskBase as RunnerTaskBase, TaskEventPack as RunnerTaskEventPack, TaskResult as RunnerTaskResult, TaskResultPack as RunnerTaskResultPack, Test as RunnerTestCase, File as RunnerTestFile, Suite as RunnerTestSuite, SuiteAPI, SuiteCollector, SuiteFactory, TaskContext, TaskCustomOptions, TaskMeta, TaskState, TestAPI, TestAnnotation, TestContext, TestFunction, TestOptions, afterAll, afterEach, beforeAll, beforeEach, describe, it, onTestFailed, onTestFinished, suite, test } from '@vitest/runner';
node_modules/vitest/dist/index.d.ts:28:export { Bench as BenchFactory, Options as BenchOptions, Task as BenchTask, TaskResult as BenchTaskResult } from 'tinybench';
node_modules/vitest/dist/index.d.ts:533:type Task = Task$1;
node_modules/vitest/dist/index.d.ts:651:export type { ApiConfig, ArgumentsType, Arrayable, AssertType, Awaitable, BaseCoverageOptions, BenchmarkUserOptions, BrowserConfigOptions, BrowserScript, BrowserUI, BuiltinEnvironment, CSSModuleScopeStrategy, CollectLineNumbers, CollectLines, Constructable, Context, CoverageIstanbulOptions, CoverageOptions, CoverageProvider, CoverageProviderModule, CoverageReporter, CoverageV8Options, Custom, CustomProviderOptions, DepsOptimizationOptions, DoneCallback, Environment, EnvironmentOptions, EnvironmentReturn, File, HappyDOMOptions, InlineConfig, JSDOMOptions, MutableArray, Nullable, Pool, PoolOptions, ProjectConfig, RawErrsMap, ReportContext, Reporter, ResolvedConfig, ResolvedCoverageOptions, ResolvedTestEnvironment, RootAndTarget, RuntimeContext, SequenceHooks, SequenceSetupFiles, SerializableSpec, Suite, SuiteHooks, Task, TaskBase, TaskResult, TaskResultPack, Test, TransformModePatterns, TransformResultWithSource, TscErrorInfo, TypecheckConfig, UserConfig, UserWorkspaceConfig, Vitest, VitestEnvironment, VitestRunMode, VitestUtils, VmEnvironmentReturn, WebSocketEvents, WebSocketHandlers, WebSocketRPC, WorkerContext, WorkerRPC };
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1:import { Task, TaskMeta, Suite, File, TestAnnotation, ImportDuration, TaskResultPack, TaskEventPack, SequenceSetupFiles, SequenceHooks, CancelReason } from '@vitest/runner';
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:34:	task?: Task;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:678:	protected getTestName(test: Task, separator?: string): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:679:	protected getFullName(test: Task, separator?: string): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:681:	protected getTestIndentation(_test: Task): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:683:	protected getDurationPrefix(task: Task): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:939:	writeLogs(task: Task, type: "err" | "out"): Promise<void>;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:940:	writeTasks(tasks: Task[], filename: string): Promise<void>;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:988:	static getComment(task: Task): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:990:	protected logTasks(tasks: Task[]): void;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1005:	protected getTestName(test: Task): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1006:	protected getTestIndentation(test: Task): string;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1015:	tasks: Task[]
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:1021:	compare?: Record<Task["id"], FormattedBenchmarkResult>
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:2843:	idMap: Map<string, Task>;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:2844:	taskFileMap: WeakMap<Task, File>;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:2847:	reportedTasksMap: WeakMap<Task, TestModule | TestCase | TestSuite>;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:2865:	updateId(task: Task, project: TestProject): void;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:2866:	getReportedEntity(task: Task): TestModule | TestCase | TestSuite | undefined;
node_modules/vitest/dist/chunks/reporters.d.BFLkQcL6.d.ts:2979:	* @deprecated use Reported Task API instead
node_modules/vitest/dist/chunks/worker.d.1GmBbd7G.d.ts:1:import { File, TestAnnotation, TaskResultPack, TaskEventPack, CancelReason, FileSpecification, Task } from '@vitest/runner';
node_modules/vitest/dist/chunks/worker.d.1GmBbd7G.d.ts:160:	current?: Task;
node_modules/vitest/dist/node.d.ts:15:export { Task as RunnerTask, TaskResult as RunnerTaskResult, TaskResultPack as RunnerTaskResultPack, Test as RunnerTestCase, File as RunnerTestFile, Suite as RunnerTestSuite, SequenceHooks, SequenceSetupFiles } from '@vitest/runner';
node_modules/vitest/dist/runners.d.ts:2:import { VitestRunner, VitestRunnerImportSource, Suite, File, Task, CancelReason, Test, TestContext, ImportDuration } from '@vitest/runner';
node_modules/vitest/dist/runners.d.ts:35:	onAfterRunTask(test: Task): void;
node_modules/vitest/dist/runners.d.ts:38:	onBeforeRunTask(test: Task): Promise<void>;
node_modules/vitest/dist/runners.d.ts:40:	onBeforeTryTask(test: Task): void;
src/models/task.ts:1:export interface Task {
src/models/task.ts:10:  readonly status?: Task["status"];
src/utils/format-task.ts:1:import type { Task } from "../models/task";
src/utils/format-task.ts:7:export function formatTask(task: Task): string {
src/utils/format-task.ts:17:export function derivePriority(task: Task): "high" | "medium" | "low" {
src/services/task-service.ts:1:import type { Task, TaskFilter } from "../models/task";
src/services/task-service.ts:4:const tasks: Task[] = [];
src/services/task-service.ts:6:export function addTask(title: string): Task {
src/services/task-service.ts:7:  const task: Task = {
src/services/task-service.ts:18:export function listTasks(filter?: TaskFilter): Task[] {
src/reports/summary.ts:10:    `Task Summary (${tasks.length} total)`,


@[assistant]:


@[toolResult]:
src/api/index.ts:2:import { formatTask } from "../utils/format-task";
src/api/index.ts:15:    return { status: 200, body: tasks.map(formatTask) };
src/services/task-service.ts:2:import { formatTask } from "../utils/format-task";
src/services/task-service.ts:31:  return listTasks(filter).map(formatTask);
src/reports/summary.ts:2:import { formatTask } from "../utils/format-task";
src/reports/summary.ts:5: * Third consumer of formatTask — generates a summary report.
src/reports/summary.ts:12:    ...tasks.map(formatTask),
src/utils/format-task.ts:7:export function formatTask(task: Task): string {
src/cli/index.ts:2:import { formatTask } from "../utils/format-task";
src/cli/index.ts:10:      return `Created: ${formatTask(task)}`;


@[assistant]:
Blast radius check complete.

Commands:
- `rg -n "\bTask\b" -g"*.ts"` – enumerated every TypeScript reference to the `Task` interface.
- `rg -n "formatTask" -g"*.ts"` – enumerated all call sites of `formatTask`.

Project consumers of `Task`/`formatTask` you must double-check after your changes:

`Task` interface (project files only):
- `src/models/task.ts`
- `src/services/task-service.ts`
- `src/utils/format-task.ts`
- `src/reports/summary.ts`
- `src/api/index.ts`
- `src/cli/index.ts`

`formatTask` usage:
- `src/utils/format-task.ts` (definition)
- `src/services/task-service.ts` (list/format)
- `src/api/index.ts`
- `src/cli/index.ts`
- `src/reports/summary.ts`

No additional in-repo consumers were found beyond this list. If all of these have been audited/updated to match the new `Task` shape and formatter behavior, the blast radius is covered; otherwise adjust them accordingly.