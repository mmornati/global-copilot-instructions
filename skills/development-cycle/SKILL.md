---
name: development-cycle
description: Guide user-authorized implementation through scoped work, independent review, evidence-based triage, and bounded remediation. Use when entering implementation, not for planning or explanation-only requests. Coordinator only; workers must not invoke the cycle recursively.
---

# Development cycle

## Entry and boundaries

Use this process only after the user has authorized implementation. A request
to explain, plan, compare, or review does not authorize code changes.
Respect the host's instructions, permissions, and available tools throughout.
This skill is guidance, not an executable workflow or a permission grant.
Do not launch a Dynamic Workflow unless the user explicitly requests it or an
applicable skill or slash command specifically instructs you to run one.
Do not install extensions, commit, push, or create pull requests without the
required authorization.

Only the coordinator runs this cycle. Mark delegated work as a worker
assignment; workers execute their bounded task and return a handoff rather
than delegating again or restarting the cycle.

## Scope and model routing

Read the actual repository guidance, relevant files, and current diff. Establish
the requirements, acceptance criteria, ownership of existing changes, and
appropriate checks before editing.

Keep small, continuous tasks in the coordinator when delegation would add more
overhead than value. Delegate only substantial, self-contained work that
benefits from separate context and is permitted by the host. Parallelize only
independent tasks with disjoint file ownership or isolated workspaces; shared
files require sequential writers. The coordinator owns integration.

For implementation and remediation, use the least expensive suitable agent
and model allowed by the user's configuration. Routine, well-scoped work can
use a lightweight worker; ambiguous behavior or cross-cutting changes need
stronger reasoning. Follow runtime defaults unless explicit user or persistent
configuration authorizes a model override. Check currently available models
rather than guessing identifiers or copying an earlier session's selection.
Escalate for demonstrated difficulty or failed attempts, not just file count.

For independent review, prefer an available Opus model when the user has
configured that preference and the host permits it. Otherwise use the runtime
reviewer default. If an explicitly required model is unavailable, report the
limitation and obtain approval for a substitute; do not silently claim the
requested model ran.

## Implementation and verification

Implement only the agreed scope, preserving unrelated user changes. Give each
worker a bounded objective, acceptance criteria, owned paths, constraints, and
stop condition. Provide only the context needed for that assignment.

Run the smallest relevant existing tests, lint, or build covering the change.
Investigate failures before fixing them; do not invent successful results.
Documentation-only work needs no build unless documentation checks exist.

## Independent review and triage

When delegation is warranted and permitted, use a separate read-only reviewer.
The reviewer must inspect the actual combined diff and necessary surrounding
code, not merely the implementer's summary. Review against the requirements
and identify regressions, integration gaps, and missing relevant coverage.

Require each finding to include severity, file and line, concrete evidence,
impact, and a suggested correction. Triage each finding against actual code:
accept, reject with evidence, or mark unresolved with the missing evidence.
Do not blindly implement speculative or out-of-scope findings.

If a separate reviewer is unavailable or disproportionate for a small task,
inspect the diff directly and disclose that review was not independent.
Never present a coordinator's self-review as an independent review.

## Remediation and stop condition

Fix accepted findings in a bounded task, selecting an appropriate worker only
when useful. Rerun the affected checks and review the resulting combined diff.
Use the independent reviewer for re-review when available and permitted.

Default to one initial review plus one remediation/re-review cycle. If material
findings remain or new ones appear at that limit, report them and request
direction instead of looping or declaring success. Do not raise a user's
budget, time, or concurrency limits without authorization.

## Compact handoff and completion

Each worker returns:

- Requirements addressed and files changed (or inspected for read-only work).
- Decisions and evidence, including commands run and actual outcomes.
- Findings with status and any unresolved risks or blockers.

The coordinator verifies the integrated result against the acceptance criteria
and provides a concise outcome, review status, and unresolved limitations.
Mention checks when requested or needed to explain a limitation; never imply
unperformed checks passed. Do not claim lower token use or cost without
comparable measurements of the whole cycle, including review and retries.
