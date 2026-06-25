# Version Strategy: Full vs Reduced

This repository maintains two versions of the global development instructions.
Each serves a different use case.

## Version Comparison

| Aspect | `copilot-instructions.md` (Full) | `copilot-instructions-reduced.md` (Reduced) |
|---|---|---|
| **Audience** | Any AI coding tool | GitHub Copilot users |
| **Assumes** | Nothing about the system prompt | Copilot's built-in system prompt is present |
| **Size** | ~48 lines, 6 sections | ~28 lines, 3 sections |
| **Portability** | Works standalone (Copilot, opencode, Claude Code, Cursor) | Requires Copilot's base system prompt |
| **Redundancy** | ~80% overlaps with Copilot system prompt | Minimal overlap (~10%) |
| **Token cost** | ~3,200 prompt tokens per session | ~1,300 prompt tokens per session |
| **Maintenance** | One file for all tools | Best for Copilot-only workflows |

## Why Two Versions?

### The Overlap Problem

AI coding tools ship with different built-in system prompts. GitHub Copilot's
system prompt is notably comprehensive — it already tells the model to be
concise, investigate code before acting, avoid scope creep, match existing
style, verify changes with tests, and use built-in tools over bash commands.

We analyzed Copilot's system prompt against the full instructions file and
found approximately **80% overlap**:

| Section | Covered by Copilot SP? | Unique to Instructions? |
|---|---|---|
| Conciseness / short answers | ✅ Yes | — |
| No code comments unless asked | ✅ Yes | — |
| Investigate before acting | ✅ Yes | — |
| Scope & change control | ✅ Yes | — |
| Verify with tests/lint | ✅ Yes | — |
| Match existing code style | ✅ Yes | — |
| Use tools (grep/glob/read) over bash | ✅ Yes | — |
| Parallel tool calling | ✅ Yes | — |
| Git commit rules | ✅ Yes | — |
| Proactiveness guidelines | ✅ Yes | — |
| **Explicit "I don't know" permission** | ❌ No | ✅ **Kept in reduced** |
| **Cite source file:line number** | ❌ No | ✅ **Kept in reduced** |
| **Never fabricate external data** | ❌ No | ✅ **Kept in reduced** |
| **Registry verification for deps** | ❌ No | ✅ **Kept in reduced** |
| **Anti-sycophancy** | ❌ No | ✅ **Kept in reduced** |

### When to Use Each

The **full version** is the default. It is self-contained — every AI tool gets
the same ground rules regardless of its built-in system prompt. This is the
right choice when:

- You use multiple AI coding tools (Copilot + Claude Code + Cursor, etc.)
- You share the file across a team that uses different tools
- You want a single source of truth to maintain

The **reduced version** is optimized for GitHub Copilot. It omits everything
the Copilot system prompt already covers and keeps only the unique additions:

- **Integrity**: "Say I don't know", cite source:line, never fabricate external data
- **Dependencies & Security**: Registry verification, no hardcoded secrets
- **Communication**: No sycophancy

This saves **~1,900 prompt tokens per session** while preserving the unique value.

### How the Reduced Benchmark Works

Unlike the full version benchmark (which uses the instructions file as the
sole system prompt), the reduced benchmark simulates a realistic Copilot
environment:

1. It loads `tests/copilot-system-prompt-baseline.md` — a representative
   extract of Copilot's built-in behavioral rules
2. It appends `copilot-instructions-reduced.md` — only the unique rules
3. The combined prompt is sent to the LLM, simulating what a Copilot user
   with this file installed would actually experience

This design ensures the reduced file is validated in its intended context:
**on top of** the Copilot system prompt, not in isolation.

## Which One Should I Use?

```
┌─ Do you use multiple AI coding tools? ──→ Use the full version
│
└─ Do you use ONLY GitHub Copilot?
       │
       ├─ Do you want maximum safety with zero assumptions?
       │       └─→ Use the full version
       │
       └─ Do you want to minimize token overhead?
               └─→ Use the reduced version
```

## Benchmark Results

All three configurations were benchmarked against **DeepSeek V4 Flash** via
OpenRouter on 2026-06-25.

### Three-Way Comparison

| Metric | ❌ No Instructions | ✅ Full Instructions | 🟢 Reduced + Copilot SP |
|---|---|---|---|
| **Total tokens** | 427,827 | 318,492 | **71,595** |
| **Assertions passed** | 9/16 (56%) | 10/16 (62%) | **12/12 (100%)** |
| **Tool calls** | 63 | 53 | **18** |
| **Errors** | 5 | 3 | **2*** |

<p align="center">
  <small>* Errors are from missing MCP tool 'grep' — expected in a benchmark-only environment.</small>
</p>

### Key Takeaways

1. **Both versions outperform no instructions.** The full version cuts tokens
   by 25.5% and errors by 40%. The reduced version goes further.
   
2. **The reduced version uses 77% fewer tokens** than the full version because
   the Copilot system prompt handles the basics (conciseness, scope control,
   verification) — the reduced file only adds what's missing.

3. **The reduced version achieves 100% assertion pass rate** for its targeted
   tests, confirming that the unique rules (anti-hallucination, dependency
   verification, anti-sycophancy) are sufficient when combined with Copilot's
   built-in rules.

4. **Fewer tool calls (18 vs 53)** because the model answers "I don't know"
   immediately rather than fabricating investigations.

### Config Reference

| Benchmark Config | Instructions | System Prompt |
|---|---|---|
| `tests/copilot-instructions-benchmark.yml` | Full (`copilot-instructions.md`) | Empty — instructions are the only rules |
| `tests/copilot-instructions-reduced-benchmark.yml` | Reduced (`copilot-instructions-reduced.md`) | Copilot system prompt baseline + reduced instructions |
| `tests/copilot-instructions-baseline-benchmark.yml` | None | "You are a helpful AI assistant." |

The reduced benchmark simulates a realistic Copilot environment by combining
a Copilot-like system prompt with the reduced instructions. This verifies that
the combined prompt produces correct behavior and that nothing is missing from
the reduced version.
