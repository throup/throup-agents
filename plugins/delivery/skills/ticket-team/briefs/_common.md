# Common block — paste whole into every brief after the role paragraph (BLIND excepted: see blind.md;
MEDIATOR takes Where and Boundaries only: see mediator.md)

## Where
Worktree: <WORKTREE> (<EITHER, the DEVELOPER: branch <BRANCH>, from <BASE_REF> OR, a reviewer:
detached at <COMMIT>, cut for you alone; nothing else changes it>). <`.env` is already in place. — OMIT
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
Who runs what: the DEVELOPER compiles (where the repo has a compile step), and runs the tests it wrote (with their mutation evidence)
and any test it needs to make a design decision, and nothing else. The REVIEWER runs the tests it
judges relevant to its range, naming what it ran and why that covers its findings. QA's rule is in
its brief. Force only the tasks you name, and confirm the forcing reached each of them (for example,
Gradle's `--rerun` applies only to the task written before it, not to a lifecycle task's
dependencies, while `--rerun-tasks` re-executes the whole graph). A partial run is reported as
partial, never as the suite passing.
Report each gate's wall time. A result replayed from a build cache is reported as a replay, not a
pass. Run every build or test command in the foreground with an explicit timeout; one that can
exceed the shell's foreground limit runs in the background, and you wait for it in the same turn
(a polling loop or a monitor). Never end your turn while one runs.
Skip set: <SKIP_PATHS, from <SKIP_SOURCE_FILE> — OR: none declared>. Gates that read skip-set
paths and never carry: <GATES, or none>. A gate result the lead says
carries is not re-run: judge the changed text instead. When you verify a fix before requesting it,
record the tree you verified (`git add -A && git write-tree && git reset -q` in your worktree,
before restoring it) and the commands you ran. When a later commit carries that fix, you may decline
to re-verify it: state that tree, `git diff --name-only <that tree> <COMMIT>` and those commands; where the
diff is empty or only skip-set paths, nothing more is needed, and otherwise re-verify in this tree
and say so. This choice covers re-verifying a fix; a gate re-run follows the lead's carry check
or, for QA, its brief.

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
mutate → run → restore → verify within one turn; never end a turn with a mutation on disk. In the
DEVELOPER's worktree, restore by re-editing the mutated lines — never `git checkout -- <file>`,
`git stash` or a copy taken earlier, which revert every uncommitted edit in the file. A reviewer's
worktree holds no uncommitted work, so `git checkout -- <file>` restores it. Verify with
`git status --short` that the tree matches its commit again.

State the commit you worked on or reviewed. Do not overwrite <DIFF_PATH>, the diff the lead wrote
for it.

## Boundaries
No push, change request or tracker write — the lead does those, and writes the commits that are
pushed. The DEVELOPER commits chunks on its branch and rebases it when the lead asks; no other role
commits. The DEVELOPER edits its worktree; a reviewer edits only its own worktree and copies in
its scratch space. Messages that reach you mid-task are findings or decisions relayed verbatim; the
lead answers no questions. If blocked, state the assumption you made and continue.
