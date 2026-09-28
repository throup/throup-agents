# throup-agents

A personal marketplace of [Claude Code](https://claude.com/claude-code) plugins for delivering
software with agents.

## Install

```
claude plugin marketplace add throup/throup-agents
claude plugin install delivery@throup-agents
claude plugin install tickets@throup-agents
```

Inside a session, the same commands work as `/plugin marketplace add throup/throup-agents` and
`/plugin install delivery@throup-agents`. Each plugin installs on its own; take the ones you want.

To update:

```
claude plugin marketplace update throup-agents
claude plugin update delivery@throup-agents
claude plugin update tickets@throup-agents
```

## Plugins

| Plugin | What it does | Docs |
|---|---|---|
| [`delivery`](plugins/delivery/README.md) | Drive a ticket to a draft change request with a sub-agent team, and hand a job over to a fresh session | [README](plugins/delivery/README.md) |
| [`tickets`](plugins/tickets/README.md) | Write a ticket that a product owner can prioritise from its visible text and an implementer can start from its collapsed block | [README](plugins/tickets/README.md) |

## Examples

From a real `delivery` run on [throup/triangles](https://github.com/throup/triangles), run with the
author's personal guidelines. While a team of agents moved that project's build to JDK 25, the
BLIND reviewer, given the diff and the code but no ticket, reported:

> 1. **Non-blocking: `mvn site` is broken by this change, and the reporting section was not
>    updated.** `./mvnw -B -DskipTests clean verify site` exits 1. …
>    - The same command with `-Dmaven.compiler.release=8` gives `BUILD SUCCESS`, so the
>      `release=25` bump is the cause.
>    …
>    - No workflow or document runs `site`, which is why I rate it non-blocking. …

More, from a ticket through a review on GitHub, a handover and the re-review round, in the
[`delivery` README](plugins/delivery/README.md#examples).

## Feedback

Open an issue on [the repository](https://github.com/throup/throup-agents/issues).

## Licence

[MIT](LICENSE).
