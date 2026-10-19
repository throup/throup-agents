# skills

One skill for keeping a skill short: it cuts the text to what the skill is for, and has a fresh
reviewer who never saw the audit judge the cut.

Built for Claude Code. Works on a skill in `~/.claude/skills/` or in a repo, and on a plugin's
skill through a checkout of the plugin's source repo. For a skill kept in version control it works
with any tracker and code host.

## Skills

### `skills:skill-audit`

Asks you to confirm, in two or three sentences, what a skill is for, then cuts every sentence that
purpose does not need and every sentence your guidelines, the repo's instruction files or the
memory already state. A blind reviewer, launched with no context from the audit, says what the cut
is for before it is shown the confirmed purpose and the original, then judges the cut against them.
Confirmed findings are applied, and a new blind reviewer checks every edit made after the review;
its findings, conflicts with your guidelines and disputed findings come back to you. An untracked
skill is backed up and edited in place; a tracked one goes through a ticket and a change request.

**Use it** when a skill has grown, when it and your guidelines disagree, or when you want a skill
trimmed. **Not for** writing a skill from nothing, or for a rule file or a prompt.

## Prerequisites

- Claude Code with sub-agents (the blind reviewer is one) and `SendMessage`, which sends the second
  stage of the brief to the same reviewer.
- For a skill kept in a repo: git, and CLI or API access to your tracker and code host,
  authenticated.

### Recommended

- [`delivery:handover`](../delivery/README.md#deliveryhandover) from this marketplace. With it,
  the first stage of the brief to the blind reviewer is a handover file that the reviewer receives,
  and closes after the second stage. Without it, the skill writes a plain brief instead and says in
  its report that `handover` is recommended.

## Customisation

The skill follows your personal guidelines (`~/.claude/CLAUDE.md`) and the project's
(`CLAUDE.md`, `AGENTS.md`) where they address the same thing. Put these in your guidelines:

- your tracker and code host, and the CLI for each, if you audit skills kept in a repo;
- where suggestions for a skill are recorded and decided, if you keep them somewhere;
- what "ready for human review" means, if not the minimum the skill states.

### Adjust freely

These are defaults; state your own rule and the skill follows it.

- Where an untracked skill is backed up (`~/.claude/skill-backups/<skill>/<date>/` by default).
- The register of agent-loaded text: imperative and terse, with no motivation, examples, history
  or prediction.
- Where suggested improvements to a skill go: by default, one line to you at the end of a run.

### The skill's mechanism: changing it changes what the skill is

- **The purpose is confirmed before anything is cut.** The confirmed text is the specification
  for the cut and for the review; without it, the auditor's reading of the skill becomes the
  standard it is judged by.
- **A sentence an authority already states is cut.** A rule stated twice drifts, and the skill
  is loaded on every use.
- **The reviewer is blind.** It gets the purpose, the baseline, the skill's artefacts and the
  authorities, never the audit's reasoning, and states what the skill is for from the cut alone
  before it is given the purpose, the baseline and the artefacts. Every edit after its review goes
  to a new blind reviewer, whose findings come back to you.
- **The audit changes no authority and adds no rule.** A conflict with your guidelines comes back
  to you, and a rule the skill never had is a suggestion, not part of the cut.

## Version

Semver in [`plugin.json`](.claude-plugin/plugin.json); each release has an entry in the
[CHANGELOG](CHANGELOG.md).
