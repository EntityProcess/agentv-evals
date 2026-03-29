# autopilot-dev Plugin Eval Results

**Date:** 2026-03-29
**PR:** [EntityProcess/agentv#824](https://github.com/EntityProcess/agentv/pull/824)
**Issue:** [EntityProcess/agentv#823](https://github.com/EntityProcess/agentv/issues/823)
**Plugin:** `plugins/autopilot-dev/` — phase-based delivery lifecycle (claim → explore → design → plan → implement → verify → ship)
**Commit:** `649ffc60` (feat/823-agentic-workflows-plugin branch)

## Results

| Eval | Tests | Claude CLI | Pi CLI |
|---|---|---|---|
| ad-explore | 3 | **3/3 PASS (1.000)** | **3/3 PASS (1.000)** |
| ad-verify | 4 | **4/4 PASS (1.000)** | 1/4 (0.729) |
| ad-design | 3 | 2/3 (0.917) | 0/3 (0.167) |
| ad-claim | 3 | 2/3 (0.889) | **3/3 PASS (1.000)** |
| ad-ship | 4 | 3/4 (0.917) | 1/4 (0.708) |
| **Total** | **17** | **14/17** | **7/17** |

## Targets

- **claude-cli**: Claude Code CLI v2.1.86 (claude-cli provider)
- **pi-cli**: Pi Coding Agent CLI via OpenRouter (openai/gpt-5.1-codex)
- **grader**: Gemini 3 Flash Preview (gemini-flash)

## Viewing Results

```bash
# View in AgentV Studio
agentv studio autopilot-dev/runs/<timestamp>/index.jsonl

# Or compare runs
agentv compare autopilot-dev/runs/<timestamp-1>/index.jsonl autopilot-dev/runs/<timestamp-2>/index.jsonl
```

## Reproduction

```bash
cd agentv
bun apps/cli/src/cli.ts eval run --target claude-cli --workers 1 evals/autopilot-dev/ad-explore.eval.yaml
```

Requires:
- `.env` with `GOOGLE_GENERATIVE_AI_API_KEY` (for gemini-flash grader)
- `claude` CLI installed (for claude-cli target)
- `.agentv/targets.yaml` with claude-cli and pi-cli targets configured

## Artifact Structure

Raw agentv output per run:

```
runs/<timestamp>/
├── index.jsonl
├── benchmark.json
├── timing.json
└── <evalset>/<test-id>/<target>/
    ├── input.md
    ├── grading.json
    └── timing.json
```
