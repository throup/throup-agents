# Handoff messages between rounds — the lead relays, verbatim where possible, and adds nothing

## REVIEWER findings → DEVELOPER (step 3)
"REVIEWER round <N>: <count> BLOCKING, <count> non-blocking. Per finding, accept and fix in the
worktree, or push back with a reason. Then rerun the gates and report: the
per-finding decision, the round's diff, the gate output, and mutation evidence with the line that
fired for anything you claim now protects something. Still no commit." <Then the findings, each as
the REVIEWER wrote it, with its scenario and output.>

## DEVELOPER push-back → the role that raised the finding (step 5; REVIEWER, QA or BLIND alike)
"The DEVELOPER pushed back on finding <N>: <push-back verbatim>. Accept the push-back, or contest
it with the specific defect that remains. If you contest, state your position so it can be handed to
a MEDIATOR as written."

## Fixed round → REVIEWER re-check (step 8)
"The DEVELOPER's per-finding decisions, verbatim: <paste>. Re-check at the current worktree state
(uncommitted; this round's diff at <DIFF_PATH>): confirm each fix, confirm nothing else changed, and
report any remaining BLOCKING finding or that the round is clear for QA and BLIND."

## Tree change after QA/BLIND passed → QA and BLIND (step 9 again, after the REVIEWER re-check)
To QA: "The tree changed after your pass and the REVIEWER has re-checked it. The whole change
against its base is now at <DIFF_PATH>. The DEVELOPER's decisions on your findings, verbatim:
<paste>. Accept or contest each decision, then review the whole change again and report as before."
To BLIND: "The change has been revised. The whole change is now at <DIFF_PATH> and a fresh copy of
the tree at <COPY_PATH>. Review it as before and report Part 1, Part 2, `shasum <DIFF_PATH>` and a
one-line verdict."

## QA or BLIND BLOCKING findings → DEVELOPER (step 10)
"<QA|BLIND> reported <count> BLOCKING, <count> non-blocking. Per finding, accept and fix in the
worktree, or push back with a reason; a push-back goes back to <QA|BLIND>. Then rerun the gates and
report as in a REVIEWER round. Still no commit." <Then the findings, each as its author wrote it.>

## MEDIATOR decision → both sides (step 7)
"The MEDIATOR decided finding <N>: <decision and reason, verbatim>. Both sides take it. <To the
DEVELOPER: fix accordingly and report as in a REVIEWER round.>"

## Blocking round fixed → QA and BLIND again (after the REVIEWER re-check)
Send both "Tree change after QA/BLIND passed" messages above, to QA and to BLIND, whichever raised
the finding, with "your pass" read as "your review".

## Cross-team relay (a downstream DEVELOPER's report about its base)
"A sibling DEVELOPER, building on your branch as its base, reports: <verbatim>. Treat it as a
reviewer finding: verify, then accept and fix or push back with a reason, and report the diff and
gate output. Do not commit."

## Post-QA/BLIND findings → DEVELOPER
"QA and BLIND passed with no BLOCKING findings. Non-blocking items follow, each as the reviewer
wrote it. Per finding: accept and fix, answer it outside the tree (the description or a follow-up), or push
back with a reason — a push-back goes back to whoever raised it. A tree change re-enters review."
