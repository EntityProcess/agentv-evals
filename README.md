# hivespec-evals

Evaluation result artifacts for [AgentV](https://github.com/EntityProcess/agentv) plugins. Each directory contains results from a specific plugin's eval suite, including console logs, JSONL indexes, grading details, and timing data.

## Plugins

| Plugin | Status | Results |
|---|---|---|
| [hivespec](./hivespec/) | PR [#824](https://github.com/EntityProcess/agentv/pull/824) | 17/17 Claude CLI, 9/17 Pi CLI |

## Reproduction

Results can be reproduced by running the eval suite in the agentv repo:

```bash
cd agentv
bun apps/cli/src/cli.ts eval run --target <target> evals/<plugin>/<eval>.eval.yaml
```
