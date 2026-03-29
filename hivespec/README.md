# hivespec Plugin Eval Results

**Date:** 2026-03-29
**PR:** [EntityProcess/agentv#824](https://github.com/EntityProcess/agentv/pull/824)
**Issue:** [EntityProcess/agentv#823](https://github.com/EntityProcess/agentv/issues/823)
**Plugin:** `plugins/hivespec/` — phase-based delivery lifecycle (claim → explore → design → plan → implement → verify → ship)
**Commit:** `390ba7b6` (feat/823-agentic-workflows-plugin)

## Results

### claude-cli (4/17 pass, 13 execution errors)

First 4 tests passed perfectly (1.000). Remaining 13 failed with `SessionStart` hook errors (Claude CLI session issue, not eval quality).

### pi-cli (10/17 pass, mean 0.824)

| Eval | Pass/Total | Details |
|---|---|---|
| hs-claim | 3/3 | All pass |
| hs-explore | 3/3 | All pass |
| hs-design | 1/3 | writes-spec passes; refuses-implementation and proposes-approaches partial |
| hs-ship | 1/4 | risk-classification passes; others partial |
| hs-verify | 2/4 | checks-blast-radius and rejects-claims pass; others partial |

## Runs

| Directory | Target | Tests |
|---|---|---|
| `2026-03-29T01-42-36-769Z` | claude-cli | 17 (4 pass, 13 errors) |
| `2026-03-29T01-42-57-395Z` | pi-cli | 17 (10 pass, 0.824 mean) |

## Viewing Results

```bash
agentv studio hivespec/runs/2026-03-29T01-42-57-395Z/index.jsonl
```

## Reproduction

```bash
cd agentv
agentv eval run --target pi-cli --workers 1 "evals/hivespec/*.eval.yaml"
```
