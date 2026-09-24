# throup-agents

A personal marketplace of [Claude Code](https://claude.com/claude-code) plugins for delivering
software with agents.

## Install

```
claude plugin marketplace add throup/throup-agents
claude plugin install delivery@throup-agents
```

Inside a session, the same commands work as `/plugin marketplace add throup/throup-agents` and
`/plugin install delivery@throup-agents`.

To update:

```
claude plugin marketplace update throup-agents
claude plugin update delivery@throup-agents
```

## Plugins

| Plugin | What it does | Docs |
|---|---|---|
| [`delivery`](plugins/delivery/README.md) | Drive a ticket to a draft change request with a sub-agent team, and hand a job over to a fresh session | [README](plugins/delivery/README.md) |

## Examples

## Feedback

Open an issue on [the repository](https://github.com/throup/throup-agents/issues).

## Licence

[MIT](LICENSE).
