---
name: handover
description: Write, place, receive and close a handover file that carries one job from this session to a fresh one. Use when a fresh session would judge better than this one — about to evaluate work against a spec or tickets this session wrote, the session deliberately left something untested, switching project or repo, the task changing kind (authoring to executing, building to reviewing), or correcting yourself repeatedly on the same class of error — when the user asks for a handover, or when starting a session that was pointed at one. Not for notes to yourself within a session.
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Handover

One file carries one job to one fresh session. The receiving session is a fresh instrument by
design; the file tells it what it cannot derive for itself and nothing else. Its judgements are
the reader's to make: state the question and the evidence, never the writer's leaning.

## When to hand over

An agent does not notice when accumulated context has become a liability. From the inside it reads
as competence: it knows the repo, the tickets, the gotchas, so continuing feels efficient. The cost
is knowing too much, and that produces no signal. So treat each of these as a trigger to say so and
offer a fresh session, rather than waiting to feel the need:

- **About to evaluate work against a spec this session helped write.** Reviewing an agent's run
  against tickets you authored, or judging output against requirements you shaped, means a vague
  answer reads as correct because you know what it was meant to say. This is the strongest
  trigger: the test measures the artefact, and you are no longer a clean instrument.
- **The session deliberately left something untested.** If you and the user agreed not to
  pre-empt a discovery, a continuing session cannot un-know it.
- **Switching project or repo** after long work elsewhere, where the previous project's
  conventions would bleed across.
- **The task changes kind**, from authoring specs to executing them, or from building to
  reviewing. Different jobs want different priors.
- **Correcting yourself repeatedly on the same class of error.** Not a context problem in itself,
  but a sign the working assumptions have drifted.

Frame it as a recommendation with the reason, not a request for permission, and name what should
carry over: memories, tickets, change-request state, open decisions. The user decides; continuing
is often right when the work is genuinely continuous. When the user accepts, write the file.

## Where the file lives

Store: `~/.claude/handovers/<yyyy-mm-dd>-<project>-<scope>.md`, or the directory the user's or the
project's guidelines name instead; outside every worktree, purged by hand and never by a session.

Pointer: `HANDOVER-<scope>.local.md` in the checkout the reader will open, a symlink to the store
file. Exclude it through the repo's exclude file (`git rev-parse --git-path info/exclude`, which
resolves from a worktree too) with the line `/*.local.md`, never through `.gitignore`; check the
line is present before creating the link. The symlink is what makes a
handover active: a store file with no symlink is history.

## What goes in

Start from `template.md` beside this file. Job, Why a fresh session, State and Verify are always
present; delete any other section that does not apply, and leave no empty heading.

1. **Job.** One sentence: what is different when the session is done. Where the artefact goes
   through review, the job ends when it is ready for human review (below). An act another party
   owns (a merge, a deploy, a tracker transition) names that party, and the session stops at the
   handoff.
2. **Why a fresh session.** Dated. What the writer shaped, briefed or wrote, so the reader knows
   which judgements are not fresh. Where the writer wrote the specification the job is measured
   against, name the criterion that rests on the writer's single reading and say to re-derive it.
3. **State.** Facts the reader will act on, each with a locator; a table where there are several.
   Each fact is as read from the live source at a stated time, not recalled. A count that decays
   while the file waits is not stated; the check that replaces it is.
4. **Open.** What is unfinished, unverified or unreviewed. Each item names a thing and what
   settles it. A claim about behaviour nothing has executed is Open, not State. A settling
   check is run against the current tree before it is written down, with an input that reaches
   the case it asks about, and the item carries what it returned. Before calling an item a
   judgement or run-dependent, name the command that would settle it and run it where it costs
   one command; an item whose settling act is still a human judgement names whose.
5. **Verify.** Commands from where the reader will be, with every override a worktree needs,
   pasted from the commands that produced the stated result. Say whether the steps are
   ordered. Say of each check whether it reads the files the job changes; one that passes on the
   untouched tree is a control, and Verify then opens with what constitutes evidence.
6. **Withheld.** What the reader must not read, be told, or do, and why. An action left to the
   user goes here with the check that replaces it.
7. **Tools.** Which skills fit this job and which do not, with the distinguishing case.

Write only what the reader cannot derive from the repo, the tracker, the change requests, the
project memory directory and the user's and the project's guidelines (`~/.claude/CLAUDE.md`, the
repo's `CLAUDE.md` and `AGENTS.md`). A pointer beats a restatement. No
narrative of how the work went.

## Ready for human review

The user's or the project's guidelines may define this; where they do not, a change request is ready when all of
these hold, each checked at its current head:

1. Every review comment is addressed, and the fix verified at the head, not assumed.
2. Every relevant commit is pushed.
3. A fresh review by an agent that did not write the change found nothing blocking. A review of an
   earlier head does not count. Where the change rests on how something else behaves, two
   reviewers ran in parallel, one of them given the diff alone.
4. Every review conversation the change addressed is marked resolved on the code host, review
   summaries and top-level comments included, or hidden as resolved where the host cannot resolve
   them; where the host can do neither, the report names the conversation and the point holds.
5. CI is green.
6. Where the job took a ticket, it is assigned to the user and in the tracker's in-progress state
   (or the label or field the guidelines name for it), or another state not yet done, as read from
   the tracker at the time.

Report the list per change request, point by point. The stopping condition is no new blocking
finding, not a review that finds nothing.

## Independence

One handover names no other session and no sibling handover. Where two pieces of work are
coupled, the coupling goes into the shared record (the change request, the ticket) and the
handover says "after X, re-read that record". Where one session writes several, grep each finished
file for the others' identifiers and paths, expecting no hit outside its own.

## Receiving

Read only the pointer you were given. Before acting, check the file's claims against the live
state; a handover is the previous session's report and a hypothesis like any other. Where it is
wrong, correct the live record you act on, not the handover.

## Closing

The session that received the file closes it, when the job is done or superseded:

1. Append `## Closed` to the store file: date, outcome in one line, and what the handover got
   wrong or left out.
2. Remove the symlink. Do not delete the store file.

A revision this skill's use suggests, at writing or at closing, goes through the channel the
user's guidelines name for skill feedback, else to the user as one line in the outcome message,
naming the store files whose `## Closed` notes show the shape. Where there is none, the message
says so. Nothing is applied to this file at
closing.

A symlink whose store file has a `## Closed` section is debris; another session names it as
debris and stops.
