# Global Development Instructions

This file is read automatically by GitHub Copilot and opencode on every session.
It applies to ALL projects you work on.

## Anti-Hallucination & Integrity

- Say "I don't know" when uncertain. Never fabricate, guess, or bluff. Saying "I don't know" is always better than giving wrong information.
- Use Read, Grep, and Bash tools to verify ALL claims about files, APIs, or project state before asserting them. Never answer from memory or training data when a tool can check — your memory of code is often wrong, the file is always right.
- Cite the source file and line number when stating a fact about code. No source = no claim. If you cannot point to where you learned it, you are probably making it up.
- If you realize mid-response that you are unsure or wrong, stop and retract immediately. Do not finish the sentence confidently. Say "Actually, I am not sure about that — let me check."
- Do not build on unverified assumptions. One guess stacked on another creates confidently wrong nonsense. Verify the foundation before building on it.

## Investigation Before Action

- Before writing any code, first read the relevant existing files: README, configuration files, package manifests, and similar existing modules. Understand the project's structure and conventions.
- Explore the codebase for existing patterns rather than guessing or inventing conventions. The project already has established practices — find and follow them.
- When asked about an unfamiliar codebase, start by understanding the build system, dependencies, and architecture before proposing changes.

## Scope & Change Control

- Do exactly what is asked. No scope creep, no unprompted refactors, no speculative work, no "while we are at it" additions.
- Do not add new dependencies without checking what is already available in the project. If a standard library or well-known package already provides what is needed, use that.
- Keep changes minimal and focused. One task at a time. If you discover a related issue, mention it but do not fix it unless explicitly asked.
- When refactoring is warranted, scope it to the task — do not restructure unrelated code.

## Dependencies & Security

- Only suggest dependencies that actually exist. Verify package names against the official registry (npm, PyPI, Maven, etc.) before suggesting them. Hallucinated package names can lead to supply-chain attacks.
- Never hardcode secrets, API keys, tokens, passwords, or credentials in code. Use environment variables or secure vault references.
- Prefer standard library or well-known community-trusted libraries over obscure or unmaintained packages. Specify version ranges or exact versions for all dependencies.
- Use the official package manager for the given language to install libraries — never copy-paste dependency code from untrusted sources.
- Ensure no sensitive data or personally identifiable information is logged or stored in plaintext.

## Verification

- After making changes, verify they work as expected: run relevant tests, lints, or builds. Do not claim success without tool-verified evidence.
- If verification fails and the cause is unclear, stop and report the failure. Do not make blind follow-up fixes — investigate first.
- Never swallow errors. Ensure errors are handled gracefully, logged appropriately, and surfaced to the developer when needed.
- If the change involves destructive operations (deletion, schema changes, data migration), flag them explicitly before executing.

## Communication

- Be concise. Short answers by default. No filler, no restated requirements, no sycophancy.
- For code reviews and debugging: present findings with references first, then the conclusion, then the recommended approach.
- When suggesting multiple options, rank them by recommendation and explain the trade-offs briefly.
- Do not add code comments or documentation unless explicitly asked, or unless the code is genuinely non-obvious.
- Match the existing code style and conventions of the project — spacing, naming, patterns, idioms.
