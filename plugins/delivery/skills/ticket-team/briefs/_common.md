# Common block — paste whole into every brief after the role paragraph (BLIND excepted: see blind.md;
MEDIATOR takes Where and Boundaries only: see mediator.md)

## Where
Worktree: <WORKTREE> (branch <BRANCH>, from <BASE_REF>). <`.env` is already in place. — OMIT
where the gates need no local environment file>
<EITHER: Dependencies are not at their relative-path locations from a worktree. Set
<DEP_ENV_VAR>=<DEP_PATH> for every gate and use <DEP_BIN_DIR> for its binaries. If any check skips
because it cannot find a dependency, that is a failure of setup, not a pass: fix the path and rerun.
OR, self-contained repo: There is no external dependency; gates run with <TOOL> from the worktree
root.>
Read <CONVENTION_FILES> before anything else.

## Gates
<GATE_COMMANDS — one per line, as the repo's verification rule states them (or as the user gave
them where the repo states none), including any step that generates code and runs the result, each
in the form that executes every task rather than replaying a build cache (for example, Gradle's `--rerun-tasks` for a gate over the whole build)>
<EITHER: Scope: full. DEVELOPER, REVIEWER and QA run every gate as listed.
OR: Scope: affected. DEVELOPER and REVIEWER run the tests that exercise the changed code directly,
and the other gates over the modules the change touches and the modules that depend on them; name
what you ran and why it covers the change. Force only the tasks you name, and confirm the forcing
reached each of them (for example, Gradle's `--rerun` applies only to the task written before it,
not to a lifecycle task's dependencies, while `--rerun-tasks` re-executes the whole graph). A
scoped pass is reported as scoped, never as the suite passing. QA runs every gate as listed each
time it reviews.>
Report each gate's wall time. A result replayed from a build cache is reported as a replay, not a
pass.

## Evidence rules
Cite code by name, not by `file:NNN`; a number is stale after any edit above it, comment edits
included. Where a number is already in the tree, a round that edits that file re-checks or replaces
it.

Mutation: break the thing each new check claims to protect. Show by `diff` or `grep` that the
mutation landed before trusting a result. The suite must go red **at the assertion that names that
check**, and the report names the line that fired; a test that reddens on a different line has
proved nothing about the line under test. A test body whose non-final assertion does not abort it
reports only its last command's status, so a non-final assertion is fail-fast
(`|| { echo … >&2; return 1; }`, or `if cmd; then return 1; fi` for a negation). Mutate in a tree
with no untracked copy of the file under review (an editor backup is a second copy the checks will
scan) and clear bytecode caches between runs (a stale `.pyc` survives a source edit). Finish
mutate → run → restore → verify within one turn; never end a turn with a mutation on disk. Over
uncommitted work, restore by re-editing the mutated lines — never `git checkout -- <file>`,
`git stash` or a copy taken earlier, which revert every uncommitted edit in the file — and verify by
re-hashing the round's diff.

State the tree you worked on or reviewed: the commit, or for uncommitted work `shasum <DIFF_PATH>`,
the diff the lead generated for this round. Do not overwrite that file.

## Boundaries
No commit, push, change request, or tracker write — the lead does those; a stacked DEVELOPER's pre-push rebase
is the one exception. The DEVELOPER edits the worktree; every
other role edits nothing but copies it makes in its own scratch space. Work autonomously; the lead
cannot answer mid-task. If blocked, state the assumption you made and continue.
