---
name: ticket-team-next
description: Trial version of ticket-team, run only when invoked by name. Drive one ticket to a draft change request (pull or merge request), or an existing change request's review round to pushed commits and replies, with a sub-agent team — DEVELOPER, REVIEWER, QA, BLIND, and MEDIATOR as an escape hatch — iterating develop→review→fix→re-review without the lead in the loop; chunks are reviewed while the DEVELOPER carries on, and REVIEWER, QA and BLIND start together once it declares READY. Use when a ticket's correctness depends on behaviour outside its diff, or when several tickets should progress in parallel. Not for a doc fix or a rename.
disable-model-invocation: true
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
A **chunk** is a local commit the DEVELOPER makes on the ticket's branch and reports to the lead
while it carries on working. **READY** is the DEVELOPER's declaration that one chunk completes the
change, against the criteria in `briefs/developer.md`.

## Roles

| Role | Job | Gets |
|---|---|---|
| DEVELOPER | Writes the code and the execution transcript | Ticket, ACs, constraints, worktree |
| REVIEWER | Reviews each chunk, and at READY the READY chunk and the fixes it carries | The chunk range's diff and worktree, ACs, DEVELOPER's reports |
| QA | Full review of the whole change at each READY | The project's review skill, the ticket, the ACs |
| BLIND | Review from the diff alone | Diff, a copy of the tree, the dependency's source, nothing else |
| MEDIATOR | Decides a contested push-back | Both positions verbatim, the ACs |

A full team runs every role. A light team runs without BLIND, for a change whose correctness is
readable from its diff. A change that depends on how something else behaves (an engine, a
provider, a tool's enforcement boundary) takes the full team, whatever the diff's size: BLIND is
the reviewer that reads the dependency's own contract unanchored by the ticket's account of it.
The lead chooses the team size from the ticket before the run starts and tells the user which
and why. MEDIATOR is spawned only when step 5 fires.

QA uses the project's own review skill where it has one. Project skills load when a session
starts, so they resolve by name only in a session started in the target repo; where the lead's
session was not, the lead tells the user before the run starts, and the user chooses a restart
there or the no-skill review below. The lead names in QA's brief the file its session loaded the
skill from. Where it has none, QA's brief says so and
QA reviews against the ACs and the repo's convention files, establishing each finding by execution.

## The loop

1. DEVELOPER commits the work as chunks and reports each as it lands.
2. REVIEWER reviews each chunk at its commit while DEVELOPER carries on. Chunks that land during a
   review make the next review's range.
3. DEVELOPER decides each finding, accept or push back with a reason, in its next chunk report. An
   accepted fix lands in a later chunk that names the finding, and REVIEWER confirms it there.
4. The raiser accepts the push-back, or contests it; re-stating the finding is a contest. An
   accepted push-back is a non-issue.
5. A contested push-back goes to MEDIATOR while work on other code continues; code the contest
   concerns waits. Both sides take its decision.
6. DEVELOPER declares READY at a chunk.
7. At READY, REVIEWER, QA and BLIND start together at that commit.
8. A BLOCKING finding from REVIEWER voids QA's and BLIND's pass at that commit and goes to
   DEVELOPER at once, who may fix it while they finish; their findings follow when they report.
   A READY declared before they report is withdrawn when they do: its commit stays as the base
   for fixing their findings, and DEVELOPER declares READY again once it has decided them.
   Otherwise REVIEWER's findings wait for theirs.
9. Every finding from the three goes to DEVELOPER, whatever its tag, and returns to 3; a push-back
   goes to the role that raised it.
10. All three pass at one READY with no blocking issue remaining: repackage, push, open a draft
    change request.

Any change to the tree after READY, a test or a comment included, is a chunk and ends in a new
READY.

## Re-review mode

The input may be an existing branch, worktree and external review instead of a fresh ticket. The
review's findings, numbered, are the ACs; the DEVELOPER's worktree is reused, not cut; DEVELOPER
commits one chunk per finding it can deliver on its own; QA reads the review conversations where
it would read the ticket; the terminal step is the lead's shared-system writes, as in a normal run:
it pushes the round's commits, posts the replies and the summary comment, updates the change
request's description and resolves the conversations the round fixed. The whole change is
`git diff $(git merge-base <base> <commit>) <commit>`, so commits the base gained since are not
shown as reversions. REVIEWER takes chunk ranges; QA and BLIND take the whole change; each brief's
`<DIFF_PATH>` says which. The DEVELOPER's fifth deliverable becomes the replies and a summary
comment.

## The lead acts at exactly four points

Keep a run log in scratch from the start: one line per handoff, and each agent's wall time from its
completion notice.

1. **On a chunk or READY**: `git -C <worktree> log --oneline <previous>..<commit>`, to confirm the
   report describes commits that exist. Write the range's diff to scratch once, read-only
   (`git diff <previous> <commit>`), and at READY the whole change as well. For REVIEWER and QA,
   cut a worktree detached at the commit (`git worktree add --detach`), holding what the gates
   need; for BLIND, such a worktree copied without `.git` to a path carrying no ticket key, since
   `git archive` drops `export-ignore` paths and writes the hash into `export-subst` ones. Remove each
   once its report arrives. `<previous>` is the last commit REVIEWER reviewed: at the start, the
   reviewed commit in re-review mode, else the merge base. A range that lands while REVIEWER is
   reviewing waits for it; at READY, QA and BLIND start at once. A READY that lands while QA or
   BLIND is still on an earlier one is withdrawn when they report (step 8).
2. **The finding ledger**, in the run log: one line per finding with its id (the raiser's initial
   and a number running across the run: R3, Q2, B5), the commit it was raised at, and each state a
   report gives it: accepted, pushed back, contested, decided, fixed in a commit, confirmed. The
   lead records what the reports say. At READY it hands REVIEWER the ids to confirm.
3. **Repackaging and writes to a shared system**: the pushed commits, the push, the change request,
   the tracker; in re-review mode also the replies, the summary comment and resolving the
   conversations the round fixed.
4. **The change request's description**, once all three have passed: the lead reads the diff
   and writes it from the DEVELOPER's and QA's independent drafts, naming the head QA's run of the
   listed gates covered where it is not the pushed head.

Nothing else: no re-running gates, no reading diffs, no re-deriving a finding, no view of the code.

## Repackaging

Chunks are the team's snapshots, not the pushed history. Before the push the lead rebuilds the
branch from `<start>` as signed commits. `<start>` is the merge base, or in re-review mode the
change request's head as pushed (`git fetch`, then `git rev-parse origin/<branch>`, recorded as a
hash because the push moves the ref), so commits already pushed
keep their hashes and the push stays a fast-forward. Before rebuilding,
`git merge-base --is-ancestor <start> HEAD` succeeds; where it fails, the lead stops and asks the
user. By
default one commit whose message the lead writes (`git reset --soft <start>`, then
`git commit -S`); where the ACs are independent findings, one commit per chunk under the
DEVELOPER's subjects, each fix folded into the chunk it names
(`git rebase --autosquash --force-rebase --gpg-sign <start>`). A rebase that stops on a conflict is aborted; the lead takes
the single commit, or hands the conflict to DEVELOPER, whose resolution is a chunk and ends in a
new READY. The keep-chunks shape pushes intermediate trees no gate ran on; the description says
so. Before the push, `git rev-parse HEAD^{tree}` equals the tree of the READY commit all
three passed (for a stacked branch, of the head REVIEWER confirmed). After it,
`git log --format='%h %G?' <start>..<pushed head>` shows every commit signed as pushed.

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
message from `briefs/handoffs.md`. The evidence rules, BLIND's exclusions and the
disposition of findings after a READY are stated there and nowhere else. A reviewer whose round
report does not arrive is replaced by a fresh agent briefed from its template, not resumed.

## Isolation

One git worktree per ticket for the DEVELOPER, branched from the base, holding what the gates need
(`.env` and similar); every reviewer works in its own, cut at the commit it reviews, so nothing
changes a tree under review. Where the change's correctness depends on another codebase checked out beside the repo
(the dependency), the worktree is given its location explicitly: a worktree does not resolve a
sibling checkout by relative path.

When one ticket must name another's code, branch the second from the first's branch and open its
change request against that branch. The second DEVELOPER reads the first's output as a base to
build on, not as a change to judge; anything it reports about that base goes to the first team
through the lead. Before repackaging the first branch, the lead records its head as the old tip,
and hands it on with the stacked-rebase message in `briefs/handoffs.md`.
Once the first is repackaged, the second DEVELOPER rebases with
`git rebase --onto <first branch> <old tip>`, which replays only its own chunks, and re-runs the
gates at the brief's scope; a REVIEWER confirms at that head, and the lead then repackages the
second branch. A rebase that resolved a conflict is a chunk and ends in a new READY.

## Gate scope

The gates are the commands the repo's verification rule states (its `AGENTS.md`, `CLAUDE.md` or
contributing guide). Where the repo states none, the lead asks the user for them before briefing
anyone.

The Gates section lists each gate in the form that executes every task rather than replaying a
build cache, and states one of two scopes:

- **Full**: DEVELOPER, REVIEWER and QA run every gate as listed; REVIEWER on each range it reviews.
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
report carries a per-change-request table naming the commit each review covered and each agent's
wall time from the run log, and the finding ledger's final states. It names the gate scope, the
head QA's run of the listed gates covered, and that run's wall time as QA reported it; where that
time and the scope in force sit on opposite sides of five minutes, it says so, and the user decides
the declaration.

A revision to this skill that the run suggests goes through the channel the user's guidelines name
for skill feedback, else to the user as one line in the report with the evidence from the run.
Where there is none, the report says so.
