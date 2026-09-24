---
name: ticket-team
description: Drive one ticket to a draft change request (pull or merge request), or an existing change request's review round to pushed commits and replies, with a sub-agent team — DEVELOPER, REVIEWER, QA, BLIND, and MEDIATOR as an escape hatch — iterating develop→review→fix→re-review without the lead in the loop. Use when a ticket's correctness depends on behaviour outside its diff, or when several tickets should progress in parallel. Not for a doc fix or a rename.
disable-model-invocation: false
allowed-tools: Agent, SendMessage, Bash, Read, Grep, Glob, TodoWrite
---

# Ticket team

One lead drives one ticket to a draft change request, or one change request's review round to
pushed commits and replies, through fixed roles that develop, review, fix and re-review. The lead routes work and
makes the writes to shared systems; it reads, judges and summarises none of the work in flight. A
handoff carries no framing, question or prompt of the lead's own, and a brief quotes the ticket's
constraints as the ticket's or the user's, none the lead inferred.

Terms: the **code host** holds the repository and its review (GitHub, GitLab, Bitbucket); a
**change request** is the unit it reviews (a pull request on GitHub or Bitbucket, a merge request
on GitLab); the **tracker** holds the ticket (GitHub issues, Jira, YouTrack). The user's and the
project's guidelines (`~/.claude/CLAUDE.md`, the repo's `CLAUDE.md` and `AGENTS.md`; read any not
already in context) name which ones, the CLI or API for each, and the tracker's states. Where they
do not, the lead asks the user before the first write; the repo's remote settles the host, never
the tracker.
**Scratch** is a directory outside every worktree: the session's scratchpad where it has one.

## Roles

| Role | Job | Gets |
|---|---|---|
| DEVELOPER | Writes the code and the execution transcript | Ticket, ACs, constraints, worktree |
| REVIEWER | Light review each time there is something deliverable | Diff, ACs, DEVELOPER's report and transcript |
| QA | Full review once the team calls it ready | The project's review skill, the ticket, the ACs |
| BLIND | Review from the diff alone | Diff, a copy of the tree, the dependency's source, nothing else |
| MEDIATOR | Decides a contested push-back | Both positions verbatim, the ACs |

A full team runs every role. A light team runs without BLIND, for a change whose correctness is
readable from its diff. A change that depends on how something else behaves (an engine, a
provider, a tool's enforcement boundary) takes the full team, whatever the diff's size: BLIND is
the reviewer that reads the dependency's own contract unanchored by the ticket's account of it.
The lead chooses the team size from the ticket before the run starts and tells the user which
and why. MEDIATOR is spawned only when step 7 fires.

QA uses the project's own review skill where it has one. Project skills load when a session
starts, so they resolve by name only in a session started in the target repo; where the lead's
session was not, the lead tells the user before the run starts, and the user chooses a restart
there or the no-skill review below. The lead names in QA's brief the file its session loaded the
skill from. Where it has none, QA's brief says so and
QA reviews against the ACs and the repo's convention files, establishing each finding by execution.

## The loop

1. DEVELOPER writes code.
2. REVIEWER reviews; reports BLOCKING / non-blocking with a failure scenario and command output.
3. DEVELOPER accepts or pushes back **per finding**.
4. Accepted findings get fixed.
5. The raiser accepts the push-back, or contests it; re-stating the finding is a contest.
6. Accepted push-backs are non-issues.
7. Contested push-backs go to MEDIATOR; both sides take its decision.
8. Anything fixed returns to 2.
9. A round with no blocking issues goes to QA and BLIND in parallel.
10. Every QA and BLIND finding returns to 3, whatever its tag; a push-back goes to the role that
    raised it.
11. No blocking issues remaining: commit, push, open a draft change request.

Any change to the tree after QA has passed, a test or a comment included, re-enters at step 2.

## Re-review mode

The input may be an existing branch, worktree and external review instead of a fresh ticket. The
review's findings, numbered, are the ACs; the worktree is reused, not cut; QA reads the review
conversations where it would read the ticket; the terminal step is the lead's shared-system
writes, as in a normal run: it pushes the round's commits, posts the replies and the summary
comment, updates the change request's description and resolves the conversations the round fixed.
Two diffs exist: the round's delta against the reviewed commit, and the whole change against its
base: `git diff $(git merge-base <base> HEAD)` plus the untracked files as in the lead's first
point, so uncommitted fixes are included and commits the base gained since are not shown as
reversions. REVIEWER takes the delta; QA and BLIND take the whole change; each brief's `<DIFF_PATH>`
says which. The DEVELOPER's fifth deliverable becomes the replies and a summary comment.

## The lead acts at exactly three points

Keep a run log in scratch from the start: one line per handoff, and each agent's wall time from its
completion notice.

1. **On handoff**: `git status --short` and `git diff --stat` in the worktree, to confirm the report
   describes a tree that exists. Write the round's diff to scratch once, read-only: `git diff HEAD`
   plus `git diff --no-index /dev/null <file>` for each untracked file, so staged hunks are
   included. For BLIND, copy the tree without `.git` to a path carrying no ticket key.
2. **Writes to a shared system**: the commit, the push, the change request, the tracker; in
   re-review mode also the replies, the summary comment and resolving the conversations the round
   fixed.
3. **The change request's description**, once QA and BLIND have passed: the lead reads the diff
   and writes it from the DEVELOPER's and QA's independent drafts, naming the head QA's run of the
   listed gates covered where it is not the pushed head.

Nothing else: no re-running gates, no reading diffs, no re-deriving a finding, no view of the code.

## Shared-system writes

The user's and the project's guidelines replace any of these defaults they address. Where the user did not ask for
a ticket-team run on this ticket, the lead confirms with the user before the first write.

- **Tracker.** In a fresh run (re-review mode leaves the ticket alone), before the first artefact
  carrying the ticket's key (a branch, a worktree, a
  commit), assign the ticket to the user and move it to the tracker's in-progress state (or the
  label or field the guidelines name for it), read from the tracker's own schema rather than
  assumed; then re-read the ticket to confirm both landed. Where the ticket is assigned to someone
  else, stop and ask the user. Reading or commenting on a ticket does not take it.
- **Change request.** Opened as a draft; where the host has no drafts, the lead asks the user. In
  re-review mode, where the change request's author is not the user, the lead asks before pushing
  to its branch or posting on it.
- **Description.** Follow the repo's change-request template where it has one; content an AC
  requires in the description stays, whatever the template's length. Scale the text to the diff,
  and comments likewise: what the change does and what to scrutinise, never a restatement of the
  diff. Detail a reviewer may want but need not read goes in a separate comment, collapsed where
  the host renders collapsed blocks. Link to a comment by its absolute URL. A description that
  already exists, or grows over rounds, is re-read whole and re-scaled, not appended to.
- **Attribution.** Everything posted to the code host under the user's account says it was
  generated by an agent. A finding is credited to its source: a sub-agent's as a review finding; a
  colleague's by the name the user gave, else their handle as the code host shows it, never a name
  inferred from the handle; the user's own, from the conversation, as discussed and never as a
  colleague's.
- **Resolving.** Once a conversation's finding is fixed and verified at the pushed head, mark it
  resolved on the code host, including review summaries and top-level comments where the host
  keeps them apart from line conversations; where the host cannot resolve one, hide it as
  resolved, and where it can do neither, leave it and say so in the report.
- **Rebase.** Where the base's history has no merge commits, a branch is updated by rebasing onto
  its base and pushing with a lease, never by merging the base in. After any base update, grep the
  branch for every path, command and symbol the base changed: a clean rebase does not flag a doc
  that names a renamed file.
- **Follow-ups.** A non-blocking finding becomes a ticket only when it was observed, not merely
  shown reachable, and the lead files it once the user agrees; otherwise it is recorded in the
  description with "not observed".

## Briefs

Every brief starts from the role's template in `briefs/` beside this file, and every between-round
message from `briefs/handoffs.md`. The evidence rules, BLIND's exclusions and the post-QA
disposition of non-blocking findings are stated there and nowhere else. A reviewer whose round
report does not arrive is replaced by a fresh agent briefed from its template, not resumed.

## Isolation

One git worktree per ticket, branched from the base, holding what the gates need (`.env` and
similar). Where the change's correctness depends on another codebase checked out beside the repo
(the dependency), the worktree is given its location explicitly: a worktree does not resolve a
sibling checkout by relative path.

When one ticket must name another's code, branch the second from the first's branch and open its
change request against that branch. The second DEVELOPER reads the first's output as a base to
build on, not as a change to judge; anything it reports about that base goes to the first team
through the lead. Before the push, the DEVELOPER rebases onto the base's current head and re-runs
the gates at the brief's scope, and a REVIEWER confirms at the head that will be pushed; a rebase
that resolved a conflict re-enters at step 2.

## Gate scope

The gates are the commands the repo's verification rule states (its `AGENTS.md`, `CLAUDE.md` or
contributing guide). Where the repo states none, the lead asks the user for them before briefing
anyone.

The Gates section lists each gate in the form that executes every task rather than replaying a
build cache, and states one of two scopes:

- **Full**: DEVELOPER, REVIEWER and QA run every gate as listed.
- **Affected**: DEVELOPER and REVIEWER run the tests that exercise the changed code directly, and
  the other gates over the modules the change touches and the modules that depend on them. QA runs
  every gate as listed each time it reviews.

BLIND runs what it chooses in either scope. Affected applies only where the project memory of the
target repo's main checkout declares it with a measured wall time for the listed gates over five
minutes and the date measured. That memory is `~/.claude/projects/<path with each / and . as ->/memory/`, the
path being `git rev-parse --path-format=absolute --git-common-dir` without its `/.git`, so a
worktree resolves to its main checkout. Everything else is Full. The criterion is the same for
every project.

## Reporting to the user

Relay findings and decisions as their authors wrote them, saying which reviewer found what. The
report carries a per-change-request table naming the tree each review covered, the commit or the
hash of the round's diff, and each agent's wall time from the run log. It names the gate scope, the
head QA's run of the listed gates covered, and that run's wall time as QA reported it; where that
time and the scope in force sit on opposite sides of five minutes, it says so, and the user decides
the declaration.

A revision to this skill that the run suggests goes through the channel the user's guidelines name
for skill feedback, else to the user as one line in the report with the evidence from the run.
Where there is none, the report says so.
