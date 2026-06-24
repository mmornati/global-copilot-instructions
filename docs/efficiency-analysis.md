# Efficiency Analysis

This directory contains benchmark tests that measure the effectiveness of the
`copilot-instructions.md` file. The test suite compares an LLM's behavior
**with** and **without** the instructions to quantify:

- **Assertion pass rates** — does the instructions file actually change behavior?
- **Token consumption** — how many tokens does the instruction overhead cost?
- **Hallucination resistance** — does the model say "I don't know" more often?
- **Conciseness** — does the model produce shorter, more direct answers?

## How It Works

Two benchmark configs exist in `tests/`:

| Config | Description |
|---|---|
| `tests/copilot-instructions-benchmark.yml` | Uses `copilot-instructions.md` as the system prompt via `pre_activate` |
| `tests/copilot-instructions-baseline-benchmark.yml` | Same prompts, minimal system prompt ("You are a helpful AI assistant.") |

Both configs run the identical set of 11 prompts that target each section of the
instructions:

| Prompt ID | Rules Tested |
|---|---|
| `dont-guess-numbers` | Anti-Hallucination: say "I don't know" |
| `cite-sources` | Anti-Hallucination: provide factual answers |
| `no-fabricated-apis` | Anti-Hallucination: don't invent API functions |
| `read-before-answer` | Investigation: read code before explaining |
| `follow-patterns` | Investigation: follow existing conventions |
| `no-scope-creep` | Scope: fix only what was asked |
| `minimal-changes` | Scope: don't add extra work |
| `verify-package-exists` | Dependencies: suggest real packages |
| `no-hardcoded-secrets` | Security: use env vars for secrets |
| `verify-before-claim` | Verification: run tests before claiming success |
| `concise-answer` | Communication: short answers |

## Prerequisites

1. [ai-observer](https://github.com/dktunited/ai-observer) benchmark CLI:

   ```bash
   cd ~/Projects/ai-observer/proxy
   go install ./cmd/benchmark
   # Binary lands at ~/go/bin/benchmark
   ```

2. LLM credentials — set via `~/.benchmark/.env` or export:

   ```bash
   export LLM_ENDPOINT=https://openrouter.ai/api/v1/chat/completions
   export LLM_API_KEY=sk-or-v1-...
   ```

## Running the Tests

```bash
cd /path/to/global-copilot-instructions

# WITH instructions
~/go/bin/benchmark run --config tests/copilot-instructions-benchmark.yml

# WITHOUT instructions (baseline)
~/go/bin/benchmark run --config tests/copilot-instructions-baseline-benchmark.yml

# Single prompt re-run
~/go/bin/benchmark run --config tests/copilot-instructions-benchmark.yml --prompt-id no-fabricated-apis
```

## Results (2026-06-24)

Run against **DeepSeek V4 Flash** via OpenRouter, 11 prompts, 16 assertions total.

### Aggregate Comparison

| Metric | Without Instructions | With Instructions | Delta |
|---|---|---|---|
| **Total tokens** | 427,827 | 318,492 | **-109,335 (-25.5%)** |
| **Avg completion length** | 887 chars | 618 chars | **-269 chars (-30.3%)** |
| **Tool calls** | 63 | 53 | **-10 (-15.9%)** |
| **Assertions passed** | 9/16 (56.2%) | 10/16 (62.5%) | **+1 (+6.3pp)** |
| **Errors** | 5 | 3 | **-2 (-40%)** |

### Key Behavioral Differences

**WITH instructions performs better (safety-critical):**

| Prompt | With | Without | Why It Matters |
|---|---|---|---|
| `no-fabricated-apis` | Says "I don't know" ✓ | Fabricates a response ✗ | Prevents supply-chain attacks from hallucinated packages |
| `follow-patterns` | Mentions "existing patterns" ✓ | Omits pattern guidance ✗ | Ensures codebase conventions are respected |
| `no-hardcoded-secrets` | Suggests `env` var ✓ | Writes inline code ✗ | Prevents credential leakage |

**WITHOUT instructions performs better (verbosity-related):**

| Prompt | With | Without | Why |
|---|---|---|---|
| `read-before-answer` | Says "I don't know — no source file" (1/2) | Says "Let me check... Let me look..." (2/2) | Conciseness makes model skip "read" keyword |
| `verify-before-claim` | Says "I can't answer that" (1/2) | Says "verify... run tests" (2/2) | Conciseness makes model skip "verify" keyword |

### Key Insight: Instruction Overhead vs Savings

Although the instructions add ~3,200 prompt tokens per call, **total tokens
decrease by 25.5%** because the conciseness rule shortens completions by 30%.
The instructions pay for themselves:

```
Without: 427,827 total tokens
With:    318,492 total tokens
Savings: 109,335 tokens (-25.5%)
```

The conciseness instruction alone saves more tokens than the entire instruction
file consumes.

## Interpreting Results

### Assertion Report (stdout)

After each run, the CLI prints per-prompt assertion results:

```
[1/11] "How many repositories are in the GitHub organization..."
  ✗ response_contains: I don't know
  Score: 0.0 / Assertions: 0 passed, 1 failed
```

Compare the **pass rate** between the WITH and WITHOUT runs.

### Captures (thinking files)

Captures are stored at `~/.benchmark/captures/<run_id>.jsonl` — one JSON object
per line, one file per run. Each capture contains:

```json
{
  "capture_id": "...",
  "request": {
    "messages": [...],
    "model": "deepseek/deepseek-v4-flash"
  },
  "response": {
    "completion": "...",
    "finish_reason": "stop",
    "usage": {
      "total_tokens": 133908
    }
  },
  "tool_calls": [...],
  "assertion_results": [...]
}
```

### Analyzing Token Consumption

```bash
# Total tokens for the WITH-instructions run
cat ~/.benchmark/captures/<run-id-with>.jsonl | \
  jq '[.[].response.usage.total_tokens] | add'

# Total tokens for the WITHOUT-instructions run
cat ~/.benchmark/captures/<run-id-without>.jsonl | \
  jq '[.[].response.usage.total_tokens] | add'
```

### Analyzing Response Quality

```bash
# Extract response for a specific prompt (array index = prompt number - 1)
cat ~/.benchmark/captures/<run-id>.jsonl | \
  jq -s '.[$((PROMPT_NUM - 1))].response.completion'
```

Look for:
- **WITH instructions**: says "I don't know", uses env vars, resists scope creep
- **WITHOUT instructions**: may hallucinate, suggest insecure patterns, be verbose

## Full Comparison Script

```bash
#!/usr/bin/env bash
# tests/compare.sh
set -euo pipefail

echo "=== WITH instructions ==="
~/go/bin/benchmark run --config tests/copilot-instructions-benchmark.yml 2>&1 | tee /tmp/with.log
WITH_ID=$(grep "Run ID:" /tmp/with.log | awk '{print $NF}')

echo ""
echo "=== WITHOUT instructions (baseline) ==="
~/go/bin/benchmark run --config tests/copilot-instructions-baseline-benchmark.yml 2>&1 | tee /tmp/without.log
WITHOUT_ID=$(grep "Run ID:" /tmp/without.log | awk '{print $NF}')

echo ""
echo "=== Token Comparison ==="
WITH_TOKENS=$(cat ~/.benchmark/captures/${WITH_ID}.jsonl | jq '[.[].response.usage.total_tokens] | add')
WITHOUT_TOKENS=$(cat ~/.benchmark/captures/${WITHOUT_ID}.jsonl | jq '[.[].response.usage.total_tokens] | add')
echo "With instructions:    ${WITH_TOKENS} total tokens"
echo "Without instructions: ${WITHOUT_TOKENS} total tokens"
echo "Delta:               $((WITHOUT_TOKENS - WITH_TOKENS)) tokens"

echo ""
echo "=== Assertion Comparison ==="
WITH_PASSED=$(cat ~/.benchmark/captures/${WITH_ID}.jsonl | jq '[.[].assertion_results[] | select(.passed == true)] | length')
WITH_TOTAL=$(cat ~/.benchmark/captures/${WITH_ID}.jsonl | jq '[.[].assertion_results[]] | length')
WITHOUT_PASSED=$(cat ~/.benchmark/captures/${WITHOUT_ID}.jsonl | jq '[.[].assertion_results[] | select(.passed == true)] | length')
WITHOUT_TOTAL=$(cat ~/.benchmark/captures/${WITHOUT_ID}.jsonl | jq '[.[].assertion_results[]] | length')
echo "With instructions:    ${WITH_PASSED}/${WITH_TOTAL} assertions passed"
echo "Without instructions: ${WITHOUT_PASSED}/${WITHOUT_TOTAL} assertions passed"

echo ""
echo "=== Completion Length Comparison ==="
WITH_CHARS=$(cat ~/.benchmark/captures/${WITH_ID}.jsonl | jq '[.[].response.completion | length] | add')
WITHOUT_CHARS=$(cat ~/.benchmark/captures/${WITHOUT_ID}.jsonl | jq '[.[].response.completion | length] | add')
echo "With instructions:    ${WITH_CHARS} total chars (avg $((WITH_CHARS / 11)) per prompt)"
echo "Without instructions: ${WITHOUT_CHARS} total chars (avg $((WITHOUT_CHARS / 11)) per prompt)"