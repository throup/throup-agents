# delivery

Two skills for delivering a change with agents: one drives the work through a review team, the
other carries a job to a fresh session when the current one has stopped being a clean judge.

Built for Claude Code and git. Other agents or version control systems may work, but are not
supported.

## Skills

### `delivery:ticket-team`

A lead agent drives one ticket to a draft change request (a pull request on GitHub or Bitbucket, a
merge request on GitLab) through fixed sub-agent roles: a DEVELOPER writes the change, a REVIEWER
checks each round, then QA and a BLIND reviewer (who sees only the diff) review it in parallel,
with a MEDIATOR for a contested push-back. The team loops until no blocking finding remains; the
lead only routes messages and makes the writes to shared systems. It also runs one round of an
existing change request's external review through to pushed commits and posted replies.

**Use it** when a ticket's correctness depends on behaviour outside its diff (an engine, a
provider, a tool's enforcement boundary), or when several tickets should progress in parallel.
**Not for** a doc fix or a rename: the team costs more than the change.

### `delivery:handover`

Writes one file that carries one job from this session to a fresh one, and receives and closes it
in the fresh session. The file holds only what the reader cannot derive: the job, why a fresh
session, the live state with locators, what is open and what settles it, and the commands that
verify the result. To receive one, start a new session in that checkout and ask it to receive
`HANDOVER-<scope>.local.md` with the skill.

**Use it** when the current session is about to judge work against a spec it wrote, has
deliberately left something untested, is switching repo, is changing kind of task, or keeps making
the same class of error — or when you ask for one. **Not for** notes to the same session.

## Prerequisites

- Claude Code with sub-agents (`ticket-team` spawns them).
- git, with worktrees (`ticket-team` cuts one per ticket).
- CLI or API access to your code host and your tracker, authenticated.
- For `handover`: permission to write to the handover store. Claude Code treats `~/.claude/` as a
  sensitive location: a non-interactive run (`claude -p`) is refused writing or appending to a
  store file there, and removing a pointer that links to one, even with an allow rule,
  `acceptEdits` or `--add-dir`. Run handovers interactively, or name a store outside `~/.claude/`
  in your guidelines.
- For `ticket-team`: a verification rule in the repo (its `AGENTS.md`, `CLAUDE.md` or
  contributing guide) naming the commands that gate a change. Without one, the lead asks you.

## Customisation

Both skills follow your personal guidelines (`~/.claude/CLAUDE.md`) and the project's (`CLAUDE.md`,
`AGENTS.md`) where they address the same thing; Claude Code loads the `CLAUDE.md` files itself, and
the skills tell the agent to read `AGENTS.md`. Put these in your guidelines:

- your code host, your tracker, and the CLI for each;
- your tracker's in-progress state (or, on a tracker without states, the label or field that
  stands for it);
- the project's review skill, if it has one, for QA to use.

`ticket-team` writes under your account: it assigns and moves the ticket, pushes the branch (with a
lease after a rebase), opens the draft change request, files an observed follow-up once you agree,
and in re-review mode posts replies, updates the description and resolves the conversations it
fixed. It asks first where you did not ask for the run, where the ticket or the change request
belongs to someone else, where the host, tracker or gates are unknown, and where the host has no
draft change requests.

Gates run at full scope. To let the developer and reviewer run only the affected tests in a slow
repo, record in Claude Code's project memory for the repo that the scope is affected, with the
measured wall time of the full gates (over five minutes) and the date measured; QA still runs
everything.

### Adjust freely

These are defaults; state your own rule and the skill follows it.

- Taking a ticket: assign it and move it to in progress before the first branch or commit that
  carries its key.
- How a change request's description is written: the repo's template, scaled to the diff.
- Attribution: everything posted to the code host under your account says an agent generated it.
- Resolving review conversations once their fix is verified.
- Rebasing a branch onto its base rather than merging the base in.
- What "ready for human review" means, which ends a handover's job.
- Where the handover store lives (`~/.claude/handovers/` by default).
- Where suggested improvements to a skill go: by default, one line to you at the end of a run.

### The skill's mechanism: changing it changes what the skill is

- **The lead does not read or judge the work in flight.** A lead that summarises a finding frames
  it for the next agent, and the independent reviews stop being independent.
- **BLIND gets the diff and the code, and nothing else**: no ticket, description, author's report
  or tracker. Its value is an account of the change that no one's framing produced; where its
  account diverges from the intent, the code is not saying what it was meant to.
- **Findings are established by execution**, and a new check is proved by breaking the thing it
  protects and naming the assertion that fired. A finding that has only been read is a hypothesis.
- **Any change after QA passed re-enters review.** Otherwise the reviewed tree and the pushed tree
  differ.
- **A change that depends on how something else behaves takes the full team.** One reviewer is
  enough only where correctness is readable from the diff.
- **A handover holds only what the reader cannot derive, and states no leaning.** The fresh session
  exists to judge afresh; a file that argues a conclusion hands the old judgement over.
- **A handover's claims are checked against the live state before acting.** The file is the
  previous session's report, and it goes stale while it waits.

## Examples

## Version

Semver in [`plugin.json`](.claude-plugin/plugin.json); each release has an entry in the
[CHANGELOG](CHANGELOG.md).
