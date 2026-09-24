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

Excerpts from real runs on [throup/triangles](https://github.com/throup/triangles), a small Java
web app, run with the author's personal guidelines, which shape some of what you see. Each excerpt
is quoted as written; `…` marks a gap, and an excerpt may stop before its message ends.

### A ticket through the team

The author started [#75](https://github.com/throup/triangles/issues/75), moving the build to JDK 25,
with `/ticket-team 75` (`/delivery:ticket-team` when installed from this plugin). Before writing
anything, and after the author corrected the lead's claim that no JDK was installed, it asked what
the repo did not say:

> The tracker is GitHub issues on throup/triangles (the ticket itself says so). Issues have no
> in-progress state. What should mark #75 as taken?
>
> …
>
> The repo states no gates. Should the team use the ticket's AC 1 command?

The author chose to assign the issue and add no label, and to gate on `./mvnw -B clean verify`, the
ticket's first acceptance criterion with `clean` added.

The BLIND reviewer, given the diff and the code but no ticket or reports, ran what the REVIEWER had
only reasoned about. From its report:

> 1. **Non-blocking: `mvn site` is broken by this change, and the reporting section was not
>    updated.** `./mvnw -B -DskipTests clean verify site` exits 1. …
>    - The same command with `-Dmaven.compiler.release=8` gives `BUILD SUCCESS`, so the
>      `release=25` bump is the cause.
>    …
>    - No workflow or document runs `site`, which is why I rate it non-blocking. …

The run ended at a draft pull request, [#77](https://github.com/throup/triangles/pull/77), with the
decisions it would not make:

> **Decisions for you:**
> 1. **Fix the `site` regression here or as a follow-up?** This change is what breaks it, although
>    no workflow or document runs `site`. …
> 2. **Which follow-up issues should I file?** None are filed yet. Each of these was observed in
>    this run: …

The author left `site` as a follow-up and chose three of the proposed issues.

### A review on GitHub, and a handover

A second ticket, [#76](https://github.com/throup/triangles/issues/76), went through the same way to
[#81](https://github.com/throup/triangles/pull/81). The author reviewed it on GitHub; one comment,
on the test row for sides `0, 1, 1`:

> Why is this not right-angled?
>
> - the angle between the two sides length 1 must be 0 degrees
> - the other two angles are _difficult_ to measure, but should be equal to each other
> - the total sum of internal angles must be 180 degrees\
> => the sum of the other two angles must be 180-0=180 degrees\
> => each of those angles must be 180/2=90 degrees\
> => there are two right angles, so this is a right-angled triangle.
>
> If I am right, we need to fix this.\
> If I have missed something subtle, then so will other readers, so we need to document why.

Asked whether the re-review should run in the same session, the lead recommended a fresh one:

> I'd run the re-review in a fresh session.
>
> … The zero-side guard and the reasoning "a degenerate triangle has no area, so no right angle"
> came from this team. … This session knows every earlier argument for the guard, so it would tend
> to brief the round as defending that decision rather than re-opening it. A fresh session only has
> the threads and the code to go on, which is what a reviewer's question deserves.

The lead offered to write a handover, and the author asked for one. The handover the lead wrote
names the judgements that are not fresh:

> Every sentence the PR description and README use to justify the zero-side guard came out of that
> run. None of them was independently re-derived. Treat each as a claim to re-derive, not as
> settled.

### Receiving the handover: the re-review round

In a new session, the author said:

> Read HANDOVER-pr81-rereview.local.md and run /ticket-team in re-review mode on PR #81.

The lead checked the file against GitHub before starting the team:

> Handover's state matches live: head `9b43f28`, two unresolved threads, two empty review bodies,
> CI green, #76 assigned to `@throup`.

Once the DEVELOPER was working, the lead asked the author:

> The two review threads on PR #81 are posted as @throup. Did you write them yourself? (It decides
> how the replies credit them.)
>
> …
>
> Should I file a follow-up issue for WebApplicationDriver.confirmResult calling assertEquals with
> its arguments swapped (predates #76)?

The author answered that they wrote both, and not to file the issue yet.

The team re-derived the guard: the code's answer stood, and the reason first given for it did not.
The reply posted to the comment above:

> The code keeps `false`, but you're right that the reason wasn't written down. It is now.
>
> The step that doesn't hold is "the other two angles should be equal to each other". That assumes
> the side lengths fix the angles, and with a zero side they don't: the two corners at its ends are
> the same point. …
>
> In b14e2b8: …
> - the old "no area" test comment is replaced, since it gave the wrong reason (1, 2, 3 has no area
>   but angles of 0°, 0° and 180°);

At the end of the round, the lead reported to the author:

> PR #81's review round is finished: both of your review threads are answered and resolved, the
> fixes are pushed as `b14e2b8`, and CI passes at that commit. The PR is still a draft; leaving
> draft and merging are yours.

## Version

Semver in [`plugin.json`](.claude-plugin/plugin.json); each release has an entry in the
[CHANGELOG](CHANGELOG.md).
