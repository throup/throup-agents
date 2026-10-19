# Changelog

All notable changes to the `delivery` plugin. Versions follow [semver](https://semver.org/).

## 0.4.2

- `handover`: cut by an audit with `skill-audit`; its opening states the skill's purpose.

## 0.4.1

- `ticket-team`: cut by an audit with `skill-audit`, its briefs included; its opening states the
  skill's purpose, and who runs which gate is stated in the briefs only.

## 0.4.0

- `ticket-team`: states its stance on Claude Code's Agent Teams: run with it off; where it is on,
  every role is spawned unnamed unless the lead names a non-blind one, and BLIND is never named.
  BLIND's brief forbids obtaining another role's brief or the ticket through the team's files,
  transcripts or a message to anyone; every other role messages only the lead, and BLIND is spawned
  from an agent type without messaging tools where one is defined.
- `ticket-team`: a pushing run finishes only once the code host's checks and review bots have
  reported on the pushed head, again after marking it ready, and their findings have been through
  the loop or answered as false positives, with REVIEWER judging each false-positive claim. From
  the second pass the user decides, per finding, whether another round runs; the change request is
  opened at the first push only, and every later push rebuilds from the pushed head.
- `ticket-team`: each role in a run, and each later round, has a scratch directory of its own; a
  reviewer's worktree or copy stays until its findings are decided; a stacked branch's whole change
  is diffed against its predecessor's branch, and held from review while the predecessor is
  rewritten under it.
- `ticket-team`: a finding its raiser tags `wording` is settled by the lead in the final text, and
  no reviewer's draft reaches the DEVELOPER; a reviewer is replaced only once its report cannot
  arrive, with inputs of its own and the earlier rounds' findings (for BLIND, only what the BLIND it
  replaces received); a READY at a commit REVIEWER has seen says what its READY review is; a fix to
  confirm is handed as its raiser wrote it.
- `ticket-team`: run-log times are read from a clock; a narrowed gate keeps every root-level task;
  a named dependency version is the one the build resolves.

## 0.3.0

- `handover`: before writing, the writer searches the store for earlier handovers on the same
  job, withholds or points to each, and resolves an active one; a superseded one is the one
  handover a session closes without receiving it, with its artefacts left for the user; one the
  user does not declare superseded means no new handover. A taken store path or pointer name
  gives the new one a scope of its own.
- `handover`: a new "Before the file is finished" pass checks every section against Withheld,
  withholds a blind reader's design-history authorities, parks with the user's agreement any
  memory that would load withheld material unasked (or the handover is not written), and gives
  every artefact the reader creates a disposal. Closing disposes of them after removing the
  symlink, and reminds the user of parked memory. The template carries the new slots.
- `handover`: the pointer goes in a worktree the writer creates for the reader where other
  sessions share the checkout; Tools names every input a skill would ask the user for, asking the
  user for values that are theirs, while a write confirmation stays with the reader, outside any
  measured window; a withheld feedback channel sends the suggestion to the `## Closed` note.
- `handover`: Independence and Receiving allow pointing to an earlier handover, and the
  debris-and-stop rule applies to a session pointed at the debris.

## 0.2.1

- `handover`: Verify names any live request a check makes as a side effect, with how to block it
  and what must stay reachable.

## 0.2.0

- `ticket-team`: the DEVELOPER commits in chunks that the REVIEWER reviews while it carries on;
  at READY the REVIEWER, QA and BLIND start together, and the lead repackages the chunks into
  signed commits whose tree is the one all three passed.
- `ticket-team`: the DEVELOPER runs only its own tests and the REVIEWER the relevant ones; QA runs
  every gate once and may then skip a gate it reasons the changes cannot affect. Gate results
  carry across changes confined to the repo's declared CI skip paths. The Full/Affected gate
  scopes are removed.
- `ticket-team`: a REVIEWER BLOCKING finding no longer voids QA's and BLIND's passes; nothing is
  pushed while one is undecided.
- `ticket-team`: the run log and deliverables live in a directory that outlives the session, and
  the lead checks the branch's distance from its base before repackaging.

## 0.1.1

- README: examples from real runs of `ticket-team` and `handover` on
  [throup/triangles](https://github.com/throup/triangles).

## 0.1.0

- `ticket-team`: drive one ticket to a draft change request, or one review round to commits, with
  a DEVELOPER, REVIEWER, QA, BLIND and MEDIATOR sub-agent team.
- `handover`: write, receive and close a file that carries one job to a fresh session.
