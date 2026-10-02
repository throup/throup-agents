# Handoff messages between rounds — the lead relays, verbatim where possible, and adds nothing

## Chunk range → REVIEWER (step 2)
"Review commits <PREVIOUS>..<COMMIT> (<chunk numbers>). The diff is at <DIFF_PATH> and your
worktree at <WORKTREE>, detached at <COMMIT>. The DEVELOPER's chunk reports, verbatim: <paste>.
Confirm these fixes: <ids, or none>. Report as before."

## REVIEWER findings → DEVELOPER, mid-task (step 3; a BLOCKING finding after READY uses step 8's message)
"REVIEWER on <PREVIOUS>..<COMMIT>: <count> BLOCKING, <count> non-blocking. Decide each in your next
chunk report: accept, with the fix in a later chunk naming the finding, or push back with a reason.
Carry on with the work in hand." <Then the findings, each as the REVIEWER wrote it, with its
scenario and output.>

## DEVELOPER push-back → the role that raised the finding (step 4; REVIEWER, QA or BLIND alike)
"The DEVELOPER pushed back on finding <ID>: <push-back verbatim>. Accept the push-back, or contest
it with the specific defect that remains. If you contest, state your position so it can be handed to
a MEDIATOR as written."

## The raiser's answer → DEVELOPER (step 4)
"<ROLE> <accepted | contested> your push-back on finding <ID>: <answer verbatim>. <IF accepted: it
is a non-issue. IF contested: it goes to a MEDIATOR; do not build on the code it concerns until
the decision arrives.>"

## MEDIATOR decision → both sides (step 5)
"The MEDIATOR decided finding <ID>: <decision and reason, verbatim>. Both sides take it. <To the
DEVELOPER: land any fix in a chunk; code that waited on this decision may go ahead.>"

## READY → REVIEWER, QA and BLIND together (step 7)
The messages to REVIEWER and QA end with the carry sentence where it applies; BLIND's never does:
"Gate results carry from
<GATED_COMMIT> for <commands>: every path changed since is in the skip set. Do not re-run them."
To REVIEWER: the chunk-range message above for the READY chunk, followed by the `<IF READY: …>`
paragraph of `reviewer.md`, filled in.
To QA, first time: its brief. Again: "The DEVELOPER declared READY again at <COMMIT>. The whole
change against its base is at <DIFF_PATH> and your worktree at <WORKTREE>. The DEVELOPER's
decisions on your findings, verbatim: <paste>. Accept or contest each decision, then review the
whole change again and report as before; your brief says when you may skip re-running a check."
To BLIND, first time: its brief. Again: "The change has been revised. The whole change is now at
<DIFF_PATH> and a fresh copy of the tree at <COPY_PATH>. Review it as before and report Part 1,
Part 2, `shasum <DIFF_PATH>` and a one-line verdict."

## REVIEWER BLOCKING → DEVELOPER, at once (step 8)
"REVIEWER at <COMMIT>: <count> BLOCKING, <count> non-blocking. Nothing is pushed until each
BLOCKING finding is decided. Decide each: fix it in a chunk, which ends in a new READY, or push back
with a reason. <IF QA or BLIND are reviewing: their findings follow when they report. A READY you
declare before they report is withdrawn when they do; keep its commit as the base for fixing their
findings, and declare READY again once you have decided them.>" <Then the findings, each as the
REVIEWER wrote it.>

## Push-back stands on a finding QA stopped early for → QA (step 8)
"Your finding <ID> was <accepted as a push-back | decided for the DEVELOPER>: <verbatim>. The READY
at <COMMIT> stands. Run the checks you skipped at that commit and report as before."

## READY findings → DEVELOPER, once all three have reported (steps 8 and 9)
"Reviews at <COMMIT>: <IF REVIEWER findings not yet sent: REVIEWER <counts>;> QA <counts>; BLIND
<counts>. Per finding: accept and fix in a chunk, answer it outside the tree where it is non-blocking
(the description or a follow-up), or push back with a reason — a push-back goes back to whoever
raised it. A tree change ends in a new READY." <Then the findings, each as its author wrote it.>

## Base too far behind → DEVELOPER (Repackaging)
"The branch lacks <COUNT> of <BASE>'s commits, over the repo's limit of <LIMIT>. Rebase onto
`origin/<BASE>`, compile and run the tests you wrote, and report the new head as a chunk.
It ends in a new READY."

## Predecessor repackaged → stacked DEVELOPER
"<PREDECESSOR_BRANCH> has been repackaged; its old tip was <OLD_TIP>. Run
`git rebase --onto <PREDECESSOR_BRANCH> <OLD_TIP>`, compile and run the tests you wrote,
and report the new head. A rebase that resolved a conflict is a chunk and ends in a new READY."

## Stacked branch rebased → REVIEWER (Isolation)
"The branch was rebased onto <PREDECESSOR_BRANCH> after it was repackaged; the head is now <HEAD>,
from <OLD_HEAD> on the old tip <OLD_TIP>, in your worktree at <WORKTREE>. Run the tests you judge
relevant to what the rebase brought in, and confirm with
`git range-diff <OLD_TIP>..<OLD_HEAD> <PREDECESSOR_BRANCH>..<HEAD>` that no code line changed in
the rebase. Report as before."

## Stacked branch rebased → QA (Isolation)
"The branch was rebased onto <PREDECESSOR_BRANCH> after it was repackaged; the head is now <HEAD>,
in your worktree at <WORKTREE>. `git range-diff <OLD_TIP>..<OLD_HEAD> <PREDECESSOR_BRANCH>..<HEAD>`
shows what the rebase brought in. Review this head as a later READY under your brief's rule for
which checks you run, and report as before."

## Cross-team relay (a downstream DEVELOPER's report about its base)
"A sibling DEVELOPER, building on your branch as its base, reports: <verbatim>. Treat it as a
reviewer finding: verify, then accept and fix in a chunk or push back with a reason."
