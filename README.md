# Global Copilot Instructions

> Your AI coding assistant is powerful. These instructions make it **reliable**.

AI coding assistants hallucinate. They guess package names that don't exist. They invent APIs. They refactor without asking. They say "I've verified the fix works" when they haven't run a single command.

This is a single file — `copilot-instructions.md` — that changes all of that. Drop it in your global config once, and every AI session across every project follows the same ground rules:

- **No hallucinations** — Say "I don't know." Verify with tools. Cite sources. Retract when wrong.
- **No scope creep** — Do what's asked, nothing more. No unprompted refactors, no speculative busywork.
- **No fake dependencies** — Only suggest packages that actually exist. No slopsquatting vulnerabilities.
- **No fake verification** — Run the tests. Show the evidence. Don't claim success without proof.
- **No filler** — Short answers. Findings first. No sycophancy.

## Why This Matters

AI coding assistants output wrong code confidently. Studies show they hallucinate entire APIs, invent package names (19.7% of suggested packages in one study didn't exist), make up test results, and silently introduce security issues. The problem isn't the model's capability — it's that the model lacks explicit guardrails telling it what NOT to do.

This file provides those guardrails. Every instruction was distilled from:

- The OpenSSF Security-Focused Guide for AI Code Assistant Instructions
- Research on LLM hallucination patterns in software engineering
- Official GitHub and VS Code documentation on custom instructions
- Real-world failure patterns observed across hundreds of AI-assisted coding sessions
- Community best practices from the 28+ tools that support AGENTS.md

## What It Solves

| Problem | Without This | With This |
|---|---|---|
| **Hallucinated code** | "This function does X" — but it doesn't exist | Agent checks the actual file before answering |
| **Fake dependencies** | "Use `pipeline-utils` from PyPI" — doesn't exist | Agent verifies package names against real registries |
| **Scope creep** | Refactors 3 unrelated files while fixing a typo | Changes exactly what was asked, nothing more |
| **Fake testing** | "I verified the fix" — didn't run anything | Runs actual tests or lints before claiming success |
| **Secret leakage** | Inlines `API_KEY="abc123"` in suggested code | Uses environment variables by default |
| **Over-explanatory** | Writes 3 paragraphs about a single-line change | Short answers, no filler, no sycophancy |

## How It Works

The file lives at `~/.copilot/copilot-instructions.md` and is read automatically by:
- **GitHub Copilot** — on every chat and agent request, in VS Code, JetBrains, and GitHub.com
- **opencode** — when configured via `opencode.json`, on every session
- **Any tool** that supports reading a global instruction file

Local project instructions (like `AGENTS.md` in a repo root) layer on top and override when they conflict. This file handles the universal rules that apply everywhere; project files handle the specifics (stack, commands, architecture).

## Installation

### For GitHub Copilot

```bash
# Create the global Copilot instructions directory
mkdir -p ~/.copilot

# Copy the instructions file
cp copilot-instructions.md ~/.copilot/copilot-instructions.md
```

That's it. GitHub Copilot automatically reads `~/.copilot/copilot-instructions.md` on every chat and agent session. No restart needed.

### For opencode

```bash
# Copy the instructions file
cp copilot-instructions.md ~/.copilot/copilot-instructions.md
```

Then add this to `~/.config/opencode/opencode.json`:

```json
{
  "instructions": ["~/.copilot/copilot-instructions.md"]
}
```

(If the file doesn't exist yet, create it with `{}` as the minimal content.)

Now opencode reads the same file. One source of truth, zero duplication.

### For Claude Code

```bash
mkdir -p ~/.claude
cp copilot-instructions.md ~/.claude/CLAUDE.md
```

Claude Code automatically reads `~/.claude/CLAUDE.md` globally.

### One-Line Install for All Three

```bash
cp copilot-instructions.md ~/.copilot/copilot-instructions.md
```

Then create or update `~/.config/opencode/opencode.json` with the `instructions` field shown above.

## Pairing with Project-Level Instructions

This file sets **universal ground rules** — things that should apply to every project regardless of stack or team. For project-specific conventions (build commands, tech stack, architecture decisions, framework patterns), create an `AGENTS.md` in each project root:

```
my-project/
  AGENTS.md       # Project-specific instructions
  src/
  tests/
```

Local `AGENTS.md` takes precedence for that project and combines with the global rules. Read more at [opencode.ai/docs/rules](https://opencode.ai/docs/rules).

## The Philosophy

**Less is more.** This file is intentionally lean (~350 lines). Research shows that bloated instruction files lower task success rates by ~3% and raise API costs by 20%+. Every line here earns its place — if a linter can enforce it, it doesn't belong in instructions. If the model can infer it from reading the codebase, it doesn't belong here.

**Be specific.** "Be honest" is vague. "Say 'I don't know' when uncertain" is specific. "Avoid hallucinations" is vague. "Use Read/Grep/Bash tools to verify claims" is specific. Every instruction is written as an actionable, checkable behavior.

**One source of truth.** Instead of maintaining separate global files for every AI tool (Copilot, opencode, Claude Code, Cursor), maintain one and point the others at it. Less drift, less maintenance, better results.

## Customizing

This file is yours. Edit it freely:

- **Add personal rules:** "Always use tabs", "Prefer `pnpm` over `npm`", "Never use lodash"
- **Remove what doesn't apply:** If certain rules don't fit your workflow, delete them
- **Add organization rules:** Security policies, compliance requirements, preferred libraries

The file is designed to be modular — each section is self-contained. Feel free to reorder, extend, or trim.

## License

MIT. Use it, share it, fork it, make it better.

---

## Testing & Efficiency

This repository includes a benchmark test suite that validates the
instructions are working and measures their efficiency impact.

- **[Efficiency Analysis](docs/efficiency-analysis.md)** — how to run tests,
  compare token consumption with/without instructions, and interpret results
- **Test configs** — [`tests/copilot-instructions-benchmark.yml`](tests/copilot-instructions-benchmark.yml)
  (with instructions) and [`tests/copilot-instructions-baseline-benchmark.yml`](tests/copilot-instructions-baseline-benchmark.yml)
  (baseline without instructions)

Requires the [ai-observer](https://github.com/dktunited/ai-observer) benchmark CLI
(`~/go/bin/benchmark run --config <file>`).
