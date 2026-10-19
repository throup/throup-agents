---
name: handover
description: Write, place, receive and close a handover file that carries one job from this session to a fresh one. Use when a fresh session would judge better than this one — about to evaluate work against a spec or tickets this session wrote, the session deliberately left something untested, switching project or repo, the task changing kind (authoring to executing, building to reviewing), or correcting yourself repeatedly on the same class of error — when the user asks for a handover, or when starting a session that was pointed at one. Not for notes to yourself within a session.
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Handover

One file carries one job from this session to one fresh session that would judge it better, and
is placed where only that reader finds it; the reader receives it, checks it against the live
state, does the job and closes it with what the file got wrong or left out. The same file serves as
the brief for an isolated reviewer. It holds what the reader cannot derive and nothing else: the
question and the evidence, never the writer's leaning, so the judgements are the reader's. The user
decides whether to hand over, rules on what the reader escalates, and owns the acts the file leaves
to them.

## When to hand over

Treat each of these as a trigger to say so and offer a fresh session:

- **About to evaluate work against a spec or tickets this session wrote or shaped.** The strongest
  trigger.
- **The session deliberately left something untested**, having agreed with the user not to
  pre-empt a discovery.
- **Switching project or repo** after long work elsewhere.
- **The task changes kind**, from authoring specs to executing them, or from building to
  reviewing.
- **Correcting yourself repeatedly on the same class of error.**

Frame it as a recommendation with the reason, not a request for permission, and name what should
carry over: memories, tickets, change-request state, open decisions. The user decides. When the
user accepts, write the file.

## Where the file lives

Store: `~/.claude/handovers/<yyyy-mm-dd>-<project>-<scope>.md`, or the directory the user's or the
project's guidelines name instead; outside every worktree, purged by hand and never by a session.

Pointer: `HANDOVER-<scope>.local.md` in the checkout the reader will open, a symlink to the store
file. Where other sessions work in that checkout, the writer creates a worktree for the reader and
the pointer goes there. Exclude it through the repo's exclude file
(`git rev-parse --git-path info/exclude`) with the line `/*.local.md`, never through `.gitignore`;
check the line is present before creating the link. A store file with no symlink is history.

Before writing, search the store for earlier files on the same job, ticket or scope, closed or not:
by name, and `grep -l` for the ticket key and the scope. Name each match in Withheld where the
reader must not see it, by path and reason and never by what it says, or point to it where it
carries something the reader needs, and say which and why. A match still active (a
`HANDOVER-*.local.md` resolving to it in any checkout or worktree of the projects the job spans,
`git worktree list`) is resolved first: one with a `## Closed` section is debris, named to the user,
and writing goes on. One without is the user's to judge. Where the user declares it superseded, the
writer closes it once the new file has passed "Before the file is finished": `## Closed` with the
date and "superseded, not received" in place of what it got wrong, its symlink removed, and its
artefacts and any parked memory its Withheld names left in place and named to the user. Where not,
no new handover is written for the job. Where a match holds the new file's store path or pointer
name, the new one takes a scope of its own.

## What goes in

Start from `template.md` beside this file. Job, Why a fresh session, State and Verify are always
present; delete any other section that does not apply, and leave no empty heading.

1. **Job.** One sentence: what is different when the session is done. Where the artefact goes
   through review, the job ends when it is ready for human review (below). An act another party
   owns (a merge, a deploy, a tracker transition) names that party, and the session stops at the
   handoff.
2. **Why a fresh session.** Dated. What the writer shaped, briefed or wrote, and so which
   judgements are not fresh. Where the writer wrote the specification the job is measured
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
   untouched tree is a control, and Verify then opens with what constitutes evidence. Name any
   live request a check makes as a side effect, such as images, fonts or scripts a browser run
   loads from production or a CDN, with how to block it and what must stay reachable.
6. **Withheld.** What the reader must not read, be told, or do, and why. An action left to the
   user goes here with the check that replaces it.
7. **Tools.** Which skills fit this job and which do not, with the distinguishing case. Where the
   reader runs a skill, name every input that skill would otherwise ask the user for, such as the
   commands it checks the work with and those it may skip. A value the user decides is asked of
   the user at writing, not chosen by the writer. A confirmation of a write the reader will draft
   is not such an input: the reader still asks it, and a run being measured stops its clock while
   it waits.

Write only what the reader cannot derive from the repo, the tracker, the change requests, the
project memory directory and the user's and the project's guidelines (`~/.claude/CLAUDE.md`, the
repo's `CLAUDE.md` and `AGENTS.md`). A pointer beats a restatement. No narrative of how the work
went.

## Before the file is finished

1. **Read every other section against Withheld.** No other section has the reader see, run or
   produce what Withheld bars at that point: not a Verify command's output, a State locator, an
   Open item's settling act or a Tools skill. Where a later stage lifts a Withheld item, the item
   says from when. Where two sections still treat one artefact differently, the file says which
   governs. Repeat this step after steps 2 and 3, which add to Withheld.
2. **For a reader who judges blind, read each file offered as an authority**, the memory directory
   included, and name in Withheld any that records the design history or review threads of the
   thing under review.
3. **Check what loads into the reader's session unasked** (the project memory index and the files
   it points to, and the user's and the project's guidelines) for anything Withheld names: a change
   request, branch or file, or its content. Ask the user, saying that every session on the machine
   loses a parked file until it is restored. With agreement, move each matching memory file into
   `<store file stem>.parked/` beside the store file, and its index line into `MEMORY.md` there. A
   match in a guidelines file is the user's to resolve. Withheld bars the reader from that folder
   and names the user as who restores it after closing, and the store's hand purge spares it until
   then. Where a match stays where the
   reader loads it, the handover is not written; say so to the user.
4. **Give every artefact the reader creates, or the writer creates for it, a disposal**: removed
   at closing, or kept until a named event by a named party. That includes a worktree, branch,
   input directory or copy of a tree, wherever it lives.

## Ready for human review

The user's or the project's guidelines may define this; where they do not, a change request is
ready when all of these hold, each checked at its current head:

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

One handover names no concurrent session and no sibling handover written alongside it; an earlier
handover on the same job is named only as "Where the file lives" says. Where two pieces of work are
coupled, the coupling goes into the shared record (the change request, the ticket) and the
handover says "after X, re-read that record". Where one session writes several, grep each finished
file for the others' identifiers and paths, expecting no hit outside its own.

## Receiving

Read only the pointer you were given, and any earlier handover it points to outside Withheld.
Before acting, check the file's claims against the live state. Where it is wrong, correct the live
record you act on, not the handover.

## Closing

The session that received the file closes it, when the job is done or superseded; a writer
superseding it closes it as "Where the file lives" says:

1. Append `## Closed` to the store file: date, outcome in one line, and what the handover got
   wrong or left out.
2. Remove the symlink. Do not delete the store file.
3. Dispose of each artefact as the handover states, and add to the note any kept, until when and
   by whom. Where Withheld names parked memory, the outcome message tells the user it awaits
   restoring.

A revision this skill's use suggests, at writing or at closing, goes through the channel the
user's guidelines name for skill feedback, or, at closing, into the `## Closed` note where the
handover withholds that channel, and the outcome message says so. With no channel named, it goes
to the user as one line in the outcome message, naming the store files whose `## Closed` notes
show the shape. Where there is no revision, the message says so. Nothing is applied to this file
at closing.

A symlink whose store file has a `## Closed` section is debris; a session pointed at it names it
as debris and stops.
