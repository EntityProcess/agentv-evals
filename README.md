# hivespec-evals

Evaluation result artifacts for [HiveSpec](https://github.com/EntityProcess/hivespec), tested with [AgentV](https://github.com/EntityProcess/agentv). Each run directory contains JSONL indexes, per-test grading, timing, and response artifacts in the standard AgentV output format.

## Plugins

| Plugin | Status | Results |
|---|---|---|
| [hivespec](./hivespec/) | PR [#824](https://github.com/EntityProcess/agentv/pull/824) | 17/17 Claude CLI, 9/17 Pi CLI |

## Reproduction

Results can be reproduced by running the eval suite in the agentv repo:

```bash
cd agentv
agentv eval run --target <target> "evals/hivespec/*.eval.yaml"
```
