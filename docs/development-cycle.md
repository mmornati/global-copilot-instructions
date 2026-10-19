# Reusable development cycle

Keep the always-loaded policy short and put the implementation, review, and
remediation process in an on-demand skill. Both instruction variants contain
the same small routing section; the portable skill is
[`skills/development-cycle/SKILL.md`](../skills/development-cycle/SKILL.md).

## Install without overwriting personal settings

Install the skill for use across repositories:

```bash
mkdir -p ~/.copilot/skills
cp -R skills/development-cycle ~/.copilot/skills/
```

If that destination already exists, review its changes before replacing it.
In the CLI, run `/skills reload` and `/skills info development-cycle`, or start
a new session in the app and confirm the skill is available. Installation and
reload are manual steps; this repository does not change your personal setup.

For the instructions, compare your installed
`~/.copilot/copilot-instructions.md` with the chosen repository variant and
merge the integrity and Development Cycle sections. Do not blindly copy over
an existing customized file: preserve local environment initialization,
toolchain paths, and other machine-specific settings.

Optionally add this preference to your personal global instructions:

> For independent development-cycle reviews, use an available Opus model when
> the host permits an explicit selection. For implementation and remediation,
> use the least expensive suitable model permitted by my configuration. If a
> required model is unavailable, ask before substituting it.

Use the host's current model list, not a version-specific identifier copied
from another session. Model and delegation policies imposed by the host still
apply; this preference does not override them.

## Behavior

```text
User-authorized implementation
  -> Establish scope and acceptance criteria
  -> Implement directly or delegate bounded, independent work
  -> Verify the integrated change
  -> Read-only independent review when warranted and permitted
  -> Triage findings against evidence
  -> Fix accepted findings, verify, and re-review once
  -> Finish or report unresolved findings
```

Small tasks stay in the coordinator when delegation would add overhead. Report
that review was not independent in that case. Concurrent writers need disjoint
ownership or isolated workspaces; the combined result still needs review.
Workers must not recursively start the cycle.

Example request:

> Use the development-cycle skill to implement the approved plan. Apply my
> configured model preferences and stop after one remediation/re-review cycle
> if findings remain.

Planning, explanation, and review-only requests do not start implementation.
The routing instruction is a policy cue, not a generic "development phase
started" event or a guarantee of enforcement.

## Optional later step: executable orchestration

No extension or Dynamic Workflow is shipped or launched here. Stabilize the
skill first; later, an explicitly requested user-scoped extension could register
a workflow with ordered stages, structured handoffs, and resumable execution.
Check the installed app's current capabilities and credential eligibility.
The workflow API is experimental; a session-authored workflow is not, by itself,
a reusable cross-repository installation.

The installed authoring guide documents session, prompt, tool, and subagent
hooks, but no generic "development phase started" hook. Do not invent one or
use a prompt hook as blanket authorization to launch expensive orchestration.

Measure total usage and accepted outcomes on comparable tasks before claiming
the skill or separate agent contexts save tokens. Include coordination,
repeated reads, review, remediation, and retries in the comparison.

## Sources

- [GitHub: About agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [GitHub: Adding agent skills for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- Installed app documentation consulted: `copilot-sdk/docs/workflows.md` and
  `copilot-sdk/docs/agent-author.md`, under the app's `Contents/Resources`.
  These describe experimental workflow APIs and extension scopes/hooks; paths
  and features can vary by installation.
