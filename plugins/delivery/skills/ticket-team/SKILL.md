---
name: ticket-team
description: Drive one ticket to a draft change request (pull or merge request), or an existing change request's review round to pushed commits and replies, with a sub-agent team — DEVELOPER, REVIEWER, QA, BLIND, and MEDIATOR as an escape hatch — iterating develop→review→fix→re-review without the lead in the loop; chunks are reviewed while the DEVELOPER carries on, and REVIEWER, QA and BLIND start together once it declares READY. Use when a ticket's correctness depends on behaviour outside its diff, or when several tickets should progress in parallel. Not for a doc fix or a rename.
disable-model-invocation: false
allowed-tools: Agent, SendMessage, TaskStop, AskUserQuestion, Bash, Read, Grep, Glob, TodoWrite
---

# Ticket team

One lead takes one ticket to a draft change request, or one change request's review round to
pushed commits and replies, in which every finding has been raised, decided and confirmed at the
head, with a run log in scratch recording each handoff and finding. Fixed roles develop, review,
fix and re-review; the lead routes their work and makes the writes to shared systems, and reads,
judges, summarises and frames none of the work in flight. A handoff carries no framing, question
or prompt of the lead's own, and a brief quotes the ticket's constraints as the ticket's or the
user's, none the lead inferred. The user rules on what the run escalates to them, and owns human
review and the merge.

Terms: the **code host** holds the repository and its review (GitHub, GitLab, Bitbucket); a
**change request** is the unit it reviews (a pull request on GitHub or Bitbucket, a merge request
on GitLab); the **tracker** holds the ticket (GitHub issues, Jira, YouTrack). The user's and the
project's guidelines (`~/.claude/CLAUDE.md`, the repo's `CLAUDE.md` and `AGENTS.md`; read any not
already in context) name which ones, the CLI or API for each, and the tracker's states. Where they
do not, the lead asks the user before the first write; the repo's remote settles the host, never
the tracker.
**Scratch** is a directory outside every worktree that outlives the session, not the session
scratchpad: the one the user's guidelines name, else `ticket-team-runs/<ticket key, or the change
request's number in re-review mode>/` beside the repo's main checkout. Each role in a run has a
directory of its own in scratch (`lead/`, `developer/`, `reviewer/`, `qa/`, `mediator/`; a
replacement agent gets a new one, such as `qa-2/`), named in its brief, and writes in no other. The
first re-review round on a change request takes `<number>/`, and each later round on it a new one
(`<number>-r2/`, `<number>-r3/`). BLIND's inputs and its scratch never go in scratch, whose path
names the ticket or change request: see the lead's first point.
**The base** is the branch the change request targets: the default branch, or for a stacked
branch its predecessor's branch. Every merge base and whole-change diff below is taken against it.
For a stacked branch the lead records the predecessor's head it was cut from or last rebased onto
(its old tip). Where `git merge-base --is-ancestor <old tip> <commit>` succeeds and `git merge-base
--is-ancestor <old tip> <base>` fails, the predecessor was rewritten under it: the lead hands out
no review of it until the DEVELOPER rebases (the stacked-rebase message). Once the run has pushed,
and throughout re-review mode, that rebase rewrites pushed commits, and the user decides, as in
Repackaging.
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
provider, a tool's enforcement boundary) takes the full team, whatever the diff's size. The lead
chooses the team size from the ticket before the run starts and tells the user which and why.
Every sub-agent loads, at its launch, the path of the lead's working directory, the branch, status
and recent commit subjects of the checkout there, the project memory index of the repository it
belongs to (for a worktree, its main checkout's), and the user's and the project's guidelines. In a
full team, the lead checks them for the ticket key and the change request's number before the run
starts and again as it launches BLIND: in the guidelines, the memory and the commit subjects, in
the form the tracker and code host write it (`#34`, `PROJ-12`); in the path, the branch and the
status's file names, also as a bare component (`34` in `delivery-34-35`). Where either shows at the
start, it tells the user, who chooses a restart where neither shows, an edit of the memory or
guidelines file it shows in (theirs to make), or a run that goes on; where either shows at BLIND's
launch, BLIND is launched. The run's report names each such exposure to BLIND.
MEDIATOR is spawned only when step 5 fires. "All three" below means REVIEWER, QA and BLIND, or
REVIEWER and QA in a light team.

QA uses the project's own review skill where it has one. Where the lead's session was not started
in the target repo, the skill does not resolve by name: the lead tells the user before the run
starts, and the user chooses a restart there or the no-skill review below. The lead names in QA's
brief the file its session loaded the skill from. Where the project has none, QA's brief says so
and QA reviews against the ACs and the repo's convention files, establishing each finding by
execution.

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
8. A BLOCKING finding from any REVIEWER review goes to DEVELOPER at once, who may fix it while QA
   and BLIND finish; their findings follow when they report. A pass holds for the tree it covered:
   a fix is a chunk and ends in a new READY, and a push-back that stands (the raiser accepts it, or
   MEDIATOR rules for DEVELOPER) leaves the READY standing; where QA had stopped early on that
   finding, it then runs the checks it skipped at the same commit.
   A READY declared before QA and BLIND report is withdrawn when they do: its commit stays as the
   base for fixing their findings, and DEVELOPER declares READY again once it has decided them.
   Otherwise REVIEWER's findings wait for theirs.
9. Every finding from the three goes to DEVELOPER, whatever its tag, and returns to 3; a push-back
   goes to the role that raised it. The exception is a non-blocking finding its raiser tags
   `wording`, about only the wording of a drafted description, reply or summary comment: the lead
   settles it when it writes the final text (point 4), and it does not go to DEVELOPER.
10. All three pass at one READY, and no BLOCKING finding is open or undecided: repackage, push,
    and at the run's first push open a draft change request.
11. A run that pushes is finished only once the code host's automated checks and review bots have
    reported on the pushed head, and again after each change the run makes to the change request's
    state (marking it ready, where the user asks). The lead expects every check the host lists or
    requires for the head, and every bot that has posted on the repo's recent change requests; it
    waits until each has reported, or for a limit it states in the run log, and the report names
    any that had not. Each finding they raise goes through the loop from step 3. A push-back
    calling one a false positive goes to REVIEWER in the raiser's place, a contest to MEDIATOR; one
    that stands, the lead posts on the finding's own conversation. A fix ends in a new READY and
    push, and this step repeats. A pass is the reports after one push or state change. In the
    first, every finding goes to DEVELOPER; from the second, every finding goes to the user first,
    who decides for each whether it goes through the loop or is declined. The lead answers a
    declined finding on its conversation as the user's decision and records it in the description,
    or files it as a follow-up under Shared-system writes.

Any change to the tree after READY, a test or a comment included, is a chunk and ends in a new
READY.

## Re-review mode

The input may be an existing branch, worktree and external review instead of a fresh ticket. The
review's findings, numbered, are the ACs; the DEVELOPER's worktree is reused, not cut; DEVELOPER
commits one chunk per finding it can deliver on its own; QA reads the review conversations where it
would read the ticket; the terminal step is the lead's shared-system writes, as in a normal run: it
pushes the round's commits, posts the replies and the summary comment, updates the change request's
description and resolves the conversations the round fixed, then follows step 11. REVIEWER takes
chunk ranges; QA and BLIND take the whole change; each brief's `<DIFF_PATH>` says which. The
DEVELOPER's fifth deliverable becomes the replies and a summary comment.

## The lead acts at exactly four points

Keep a run log in the lead's scratch directory from the start: one line per handoff, and each
agent's wall time from its completion notice. A notice after a resume reports that stint's duration
and tool uses alone; an interim notice sent while an agent waits on its own background work reports
the time since it was spawned; the log says which kind each is. The notice's token figure is not
spend: log it as given, marked as such, and take token cost from the transcripts' per-call usage
afterwards. A stop for the usage limit and a wait on the user are logged with their start and end,
and reported wall times leave them out. Every time in the log is read from a clock, never estimated,
and is the time of the event the line records: `date` as the event happens, or the event's own
timestamp (a notice's, the code host's), converted to one named zone. A line written after its event
says so. Each report REVIEWER, QA, BLIND or MEDIATOR returns is saved, as the text it returned, to a
file of its own in the lead's scratch directory, named for the role, a number running across that
role's reports, and the commit it covered where it covers one; no saved report is overwritten, and
the log names its file.

1. **On a chunk or READY**: `git -C <worktree> log --oneline <previous>..<commit>`, to confirm the
   report describes commits that exist. Write the range's diff to scratch once, read-only
   (`git diff <previous> <commit>`), and at READY the whole change as well
   (`git diff $(git merge-base <base> <commit>) <commit>`). For REVIEWER and QA, cut a worktree
   detached at the commit (`git worktree add --detach`), holding what the gates need; for BLIND,
   such a worktree copied without `.git` (not with `git archive`), together with BLIND's copy of
   the diff and a `scratch/` directory for its own files, in a directory whose path carries no
   ticket key or change-request number. Each stays in place until every finding in its report is
   decided; then remove it. `<previous>` is the last commit REVIEWER reviewed: at the start, the
   reviewed commit in re-review mode, else the merge base. A range that lands while REVIEWER is
   reviewing waits for it; at READY, QA and BLIND start at once. For REVIEWER judging a
   false-positive push-back (step 11), cut a worktree at the pushed head; for a replacement
   reviewer (Briefs), new inputs at new paths, after stopping the agent it replaces where needed.
2. **The finding ledger**, in the run log: one line per finding with its id (the raiser's initial, H
   for the code host's checks and bots, and a number running across the run: R3, Q2, B5, H1), the
   commit it was raised at, and each state a report gives it: accepted, pushed back, contested,
   decided, fixed in a commit, confirmed, settled by the lead (a `wording` finding), answered as a
   false positive (step 11), and for a fix its raiser verified, the tree it verified and whether the
   raiser re-verified. The lead records what the reports say. At READY it hands REVIEWER each
   finding to confirm as its raiser wrote it, never by id alone. For each READY the log names the
   saved report that is REVIEWER's pass or fail at that commit.
3. **Repackaging and writes to a shared system**: the pushed commits, the push, the change request,
   the tracker, collecting the findings of the host's checks and bots on the pushed head and posting
   the false-positive answers (step 11); in re-review mode also the replies, the summary comment and
   resolving the conversations the round fixed.
4. **The final text**, once all three have passed: the lead reads the diff and writes the change
   request's description from the DEVELOPER's and QA's independent drafts, naming, for any gate
   whose last QA run is not at the pushed head, the commit that run covered; in re-review mode it
   writes the replies and the summary comment from the DEVELOPER's drafts. It settles there each
   `wording` finding (step 9). No draft reaches another drafting role: a reviewer's draft stays
   out of everything the DEVELOPER is handed.

Nothing else: no re-running gates, no reading diffs, no re-deriving a finding, no view of the code.
The carry check and the base count read file names and commit counts only.

## Repackaging

Chunks are the team's snapshots, not the pushed history. Before repackaging, the lead counts the
base's commits the branch lacks (`git -C <worktree> fetch`, then `git -C <worktree> rev-list --count
HEAD..origin/<base>`). Where the repo's checks set a limit and the count is over it, DEVELOPER
updates the branch per the Rebase rule below; the rebased tree is a chunk and ends in a new READY.
Once the run has pushed, and throughout re-review mode, that rewrites pushed commits, so the lead
reports the count and the user decides. Where no limit is set, the report states the count.

Before the push the lead rebuilds the branch from `<start>` as signed commits. `<start>` is the
merge base until the run's first push; after it, and throughout re-review mode, it is the change
request's head as pushed (`git fetch`, then `git rev-parse origin/<branch>`, recorded as a hash).
Before rebuilding, `git merge-base --is-ancestor <start> HEAD` succeeds; where it fails, the lead
stops and asks the user. By default one commit whose message the lead writes (`git reset --soft
<start>`, then `git commit -S`); where the ACs are independent findings, one commit per chunk under
the DEVELOPER's subjects, each fix folded into the chunk it names (`git rebase --autosquash
--force-rebase --gpg-sign <start>`), after which `git log --format=%s <start>..HEAD` shows no
`fixup!` subject; where it does, the lead takes the single commit. A fix to a commit already pushed
stays a commit of its own. A rebase that stops on a conflict is aborted; the lead takes the single
commit, or hands the conflict to DEVELOPER, whose resolution is a chunk and ends in a new READY.
Where the keep-chunks shape pushes intermediate trees no gate ran on, the description says so.
Before the push, `git rev-parse HEAD^{tree}` equals the tree of the READY commit all three passed
(for a stacked branch, of the head REVIEWER and QA confirmed). After it, `git log --format='%h
%G?' <start>..<pushed head>` shows every commit signed as pushed.

## Shared-system writes

The user's and the project's guidelines replace any of these defaults they address. Where the user
did not ask for a ticket-team run on this ticket, the lead confirms with the user before the first
write.

- **Tracker.** In a fresh run (re-review mode leaves the ticket alone), before the first artefact
  carrying the ticket's key (a branch, a worktree, a commit), assign the ticket to the user and
  move it to the tracker's in-progress state (or the label or field the guidelines name for it),
  read from the tracker's own schema rather than assumed; then re-read the ticket to confirm both
  landed. Where the ticket is assigned to someone else, stop and ask the user. Reading or
  commenting on a ticket does not take it.
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
  branch for every path, command and symbol the base changed.
- **Follow-ups.** A non-blocking finding becomes a ticket only when it was observed, not merely
  shown reachable, and the lead files it once the user agrees; otherwise it is recorded in the
  description with "not observed".

## Briefs

Every brief starts from the role's template in `briefs/` beside this file, and every between-round
message from `briefs/handoffs.md`. The evidence rules, who runs which gate, BLIND's exclusions and
the disposition of findings after a READY are stated there and nowhere else. A reviewer's or
MEDIATOR's report is the text it returns: no brief or message asks either to write it to a file.

A reviewer is replaced only once its report can no longer arrive: it ended without one, or the lead
stopped it. A message to a finished reviewer that comes back queued resumes it, and its report is
still coming. The replacement is a fresh agent, not resumed, briefed from its template with inputs
of its own (a new worktree or copy and diff, at new paths) and with the earlier rounds' findings
and the decisions on them, verbatim. A replacement BLIND gets only what the BLIND it replaces
received: its own findings, the push-backs on them and MEDIATOR's rulings.

## Agent Teams

Run with Claude Code's experimental Agent Teams feature off (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`
unset); a session whose `Agent` tool takes a `name` has it on. Off is recommended, not required.
Every role is spawned without a name; only with the feature on may the lead name DEVELOPER,
REVIEWER, QA or MEDIATOR, and it logs which it named. BLIND is never named. Every brief but
BLIND's allows messages to the lead only; BLIND's allows none, and forbids every route to another
role's brief or the ticket. Where an agent type is defined for BLIND without messaging tools, the
lead spawns BLIND from it, with its brief unchanged. Should a message from BLIND arrive anyway, the
lead answers it with nothing BLIND's brief withholds.

## Isolation

One git worktree per ticket for the DEVELOPER, branched from the base, holding what the gates need
(`.env` and similar); every reviewer works in its own, cut at the commit it reviews. Where the
change's correctness depends on another codebase checked out beside the repo (the dependency), the
worktree is given its location explicitly. A dependency version a brief names is the one the build
resolves (its lockfile or dependency report, not a catalog's declaration), for each artifact the
change reads.

When one ticket must name another's code, branch the second from the first's branch and open its
change request against that branch. The second DEVELOPER reads the first's output as a base to build
on, not as a change to judge; anything it reports about that base goes to the first team through the
lead. Once the first branch is rewritten, the lead hands the old tip (as "The base" defines it) on
with the stacked-rebase message in `briefs/handoffs.md`, and the second DEVELOPER rebases with
`git rebase --onto <first branch, or the base once the first is merged> <old tip>`, compiles and
runs the tests it wrote; a REVIEWER confirms at that head with `git range-diff`, QA reviews that
head under its rule for which checks it runs (BLIND does not), and the lead then repackages the
second branch. A rebase that resolved a conflict is a chunk and ends in a new READY.

## Gates

The gates are the commands the repo's verification rule states (its `AGENTS.md`, `CLAUDE.md` or
contributing guide). Where the repo states none, the lead asks the user for them before briefing
anyone; `briefs/_common.md` states the form the Gates section lists them in.

A gate result belongs to the tree it ran on, and holds for a later tree whose every changed path is
in the repo's skip set: the paths its CI declares need no build (for example, the exclusions of a
paths filter that gates the build job), named with that file in each brief's Gates section; a repo
that declares none has an empty set. A comment inside a source file is never in it, and a gate that
reads skip-set paths (a docs or link check) never carries; the Gates section names such gates.
BLIND's results never carry, and no carry is named to it. The lead checks a carry from file names
alone (`git diff --name-only <gated commit> <commit>` against the set) and says in the handoff which
results carry. A carried result is not re-run, QA's included, and the report names the commit the
carried run covered. The same holds for a tree a raiser verified before requesting a fix: where a
READY's tree differs from it only in the skip set, the raiser's results from that verification
stand for the READY.

## Reporting to the user

Where a role's report stops on something only the user can do (a machine change, a credential, a
host file), the lead puts it to the user as a question prompt at once, not as a line in a status
update.

Relay findings and decisions as their authors wrote them, saying which reviewer found what. The
report carries a per-change-request table naming the commit each review covered and each agent's
wall time from the run log, and the finding ledger's final states. For each gate it names the
commit QA's last run of it covered, that run's wall time, and any reason QA gave for not re-running
it. It names any exposure to BLIND that the lead's working-directory check found (Roles).

A revision to this skill that the run suggests goes through the channel the user's guidelines name
for skill feedback, else to the user as one line in the report with the evidence from the run.
Where there is none, the report says so.
