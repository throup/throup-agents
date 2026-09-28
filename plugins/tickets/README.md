# tickets

One skill for writing a ticket that two readers can use: a product owner who prioritises from
the visible text, and an implementer who starts from one collapsed block.

Built for Claude Code. It reads and writes the tracker through whatever CLI or API you name, so
it works with YouTrack, Jira, GitHub Issues or another tracker; none is assumed.

## Skills

### `tickets:write-ticket`

Writes or rewrites a ticket in a fixed shape: an imperative title, two short paragraphs a product
owner can prioritise from, numbered acceptance criteria, and one collapsed block holding
everything the implementer would otherwise re-derive (mechanism, pinned locators, evidence,
options not chosen). Each sentence goes to the reader who acts on it. It also reshapes an
existing ticket, folds a scope-changing comment back into the description, merges duplicates,
and writes an umbrella over subtasks.

**Use it** when filing a ticket from an investigation, when a ticket has grown into an argument
rather than a decision, or when two tickets describe one problem. **Not for** a one-line note to
yourself: the shape costs more than the note.

## Prerequisites

- Claude Code.
- CLI or API access to your tracker, authenticated, so the skill can read a ticket's fields and
  write the result.

## Customisation

The skill follows your personal guidelines (`~/.claude/CLAUDE.md`) and the project's
(`CLAUDE.md`, `AGENTS.md`) where they address the same thing. Put these in your guidelines:

- your tracker and the CLI for it;
- the tracker's state for work that will not be done, if the skill should close duplicates,
  and the label or field that stands in on a tracker without states;
- where the repository line at the end of the collapsed block should point, if not the code host
  URL of the repo the change lands in.

The skill writes under your account. It edits only tickets you reported, asks before changing
one that belongs to someone else, and does not rewrite a resolved ticket. Where no tracker is
named, it asks before writing.

### Adjust freely

These are defaults; state your own rule and the skill follows it.

- The title length (twelve words) and the first paragraph's length (two sentences).
- The heading text of the acceptance criteria and of the collapsed block.
- The attribution line at the end of the ticket.
- How the collapsed block is rendered on a tracker whose Markdown has no `<details>`.

### The skill's mechanism: changing it changes what the skill is

- **Two readers, and a sentence goes to the one who acts on it.** Visible text that names a file
  or a commit makes the product owner read the implementer's material; a block that argues the
  priority makes the implementer read the product owner's.
- **Acceptance criteria state outcomes and constraints, not the mechanism.** An AC that
  prescribes the fix makes a design decision on the implementer's behalf, and dates as the code
  moves.
- **Locators are pinned.** "Line N on `<base>` at `<sha>`" still resolves after the file moves;
  a bare line number does not.
- **The visible text leads with the decision, not the argument.** A ticket is not an argument
  building to a conclusion; the reader wants the conclusion and the option to audit it.
- **Only the reporter's own tickets are rewritten.** Rewriting another person's ticket changes
  what they said under their name.

## Version

Semver in [`plugin.json`](.claude-plugin/plugin.json); each release has an entry in the
[CHANGELOG](CHANGELOG.md).
