# Version Strategy: Full vs Reduced

This repository maintains two versions of the global development instructions.
Each serves a different use case.

## Version Comparison

| Aspect | `copilot-instructions.md` (Full) | `copilot-instructions-reduced.md` (Reduced) |
|---|---|---|
| **Audience** | Any AI coding tool | GitHub Copilot users |
| **Assumes** | Nothing about the system prompt | Copilot's built-in system prompt is present |
| **Portability** | Works standalone (Copilot, opencode, Claude Code, Cursor) | Requires Copilot's base system prompt |
| **Evidence policy** | Inspect available tools, cite evidence, explain limits | Same |
| **Development cycle** | Optional on-demand skill routing | Same |
| **Maintenance** | One file for all tools | Best for Copilot-only workflows |

## Why Two Versions?

### The Overlap Problem

AI coding tools ship with different built-in system prompts. GitHub Copilot's
system prompt is notably comprehensive — it already tells the model to be
concise, investigate code before acting, avoid scope creep, match existing
style, verify changes with tests, and use built-in tools over bash commands.

The repository's representative baseline covers many general behaviors.
Actual built-in instructions vary by host and version; the table describes
the intended split, not a current percentage measurement:

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
| **Evidence before an answer or refusal** | Not assumed | ✅ **Kept in both** |
| **Cite source file:line number** | ❌ No | ✅ **Kept in reduced** |
| **Never fabricate external data** | Not assumed | ✅ **Kept in both** |
| **Optional development-cycle routing** | Not assumed | ✅ **Kept in both** |
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

- **Integrity**: Inspect authorized tools before assuming access is unavailable,
  cite consulted sources, distinguish retrieved facts from inference, and
  explain specific limitations when evidence cannot establish an answer
- **Dependencies & Security**: Registry verification, no hardcoded secrets
- **Development Cycle**: Load the optional skill for authorized implementation,
  without recursive worker invocation or implied workflow/publication permission
- **Communication**: No sycophancy

Uncertainty should trigger investigation, not a fixed refusal phrase. The
earlier reduced version incorrectly asserted that external access never
existed. The current policy requires checking available tools first.

Do not copy machine-specific settings into either shared variant. Merge these
rules into an existing personal global file, preserving local toolchain setup.
See [development-cycle installation and model preferences](development-cycle.md).

### How the Reduced Benchmark Works

Unlike the full version benchmark (which uses the instructions file as the
sole system prompt), the reduced benchmark simulates a realistic Copilot
environment:

1. It loads `tests/copilot-system-prompt-baseline.md` — a representative
   extract of Copilot's built-in behavioral rules
2. It appends `copilot-instructions-reduced.md` — only the unique rules
3. The combined prompt is sent to the LLM, simulating what a Copilot user
   with this file installed would actually experience

This exercises the reduced file **on top of** a representative baseline, not
in isolation. It is a simulation, not the actual app runtime. Current fixtures
cover available, missing, and denied evidence. Actual tool discovery and
orchestration need the [trace-based checks](../tests/behavior-regressions.md).

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

## Historical Benchmark Results

The repository previously reported these figures against **DeepSeek V4 Flash**
via OpenRouter on 2026-06-25. Raw captures are not included, and these are not
results for the current files. The reduced suite uses different prompts, so
cross-suite token totals are not a controlled efficiency comparison.

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

### Interpretation limits

The old assertions rewarded "I don't know" even where tools might have supplied
an answer. Fewer calls or a higher keyword pass rate therefore did not prove
better factual behavior. Current assertions no longer require that phrase or
reward an unverified claim that access is unavailable.

Use matched prompts and available tools, inspect actual traces, and compare
accepted outcomes as well as total cost. No savings are claimed for the new
skill or instruction changes without new measurements.

### Config Reference

| Benchmark Config | Instructions | System Prompt |
|---|---|---|
| `tests/copilot-instructions-benchmark.yml` | Full (`copilot-instructions.md`) | Empty — instructions are the only rules |
| `tests/copilot-instructions-reduced-benchmark.yml` | Reduced (`copilot-instructions-reduced.md`) | Copilot system prompt baseline + reduced instructions |
| `tests/copilot-instructions-baseline-benchmark.yml` | None | "You are a helpful AI assistant." |

The reduced benchmark combines a Copilot-like baseline with the reduced
instructions. It provides smoke-test coverage only, not a guarantee of correct
behavior or completeness.
