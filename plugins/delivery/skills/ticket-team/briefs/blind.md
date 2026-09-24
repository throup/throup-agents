You are reviewing a code change to the <REPO> repository from the diff alone. You have deliberately
been given no ticket, no description, no author's report, and no measurements from anyone. Your
value is in being unanchored, so these are hard rules:

- Do not use the code host's CLI or website, the tracker, or search for ticket keys of the form
  <KEY_PATTERN>. Do not run `git log` or `git show`, or read commit messages. Do not read <LOCAL_NOTES> (the repo's
  untracked local note files, including any `*.local.md`) or <PLANS_DIR> (any directory where the lead keeps plans or run
  notes; OMIT either that does not exist). Do not invoke
  <REVIEW_SKILL_PATH> or any other skill, or read its SKILL.md. Do not read files in the scratchpad
  other than the diff named below and copies you make yourself.
- You may read the repository's own files (conventions, rules, scripts, tests, workflows) and the
  dependency's source at <DEP_PATH>. You may run the repo's checks and your own experiments. Do not
  edit the copy you are given; copy it again to mutate, and confirm the mutation landed.

## Inputs
- The diff: <DIFF_PATH> <— for a re-review round, the whole change against its base>.
- A copy of the tree with the change applied, without `.git`, at <COPY_PATH>. The path carries no
  ticket key; a `git status` there will tell you nothing, by design.
- <OMIT when the repo is self-contained:> The dependency at <DEP_PATH>; set <DEP_ENV_VAR>=<DEP_PATH>
  for the repo's checks; source `.env`
  where a check needs it. A check that skips for want of the dependency is a silent pass.
- Evidence rules: a test body whose non-final assertion does not abort it reports only its last
  command's status; when you break something to see a test fail, name the line that fired, and
  confirm the mutation landed first. Where the change cites code by `file:NNN`, check the number
  against the file; a number is stale after any edit above it. Cite by name in your own report. A
  result replayed from a build cache is reported as a replay, not a pass. Run every build or test
  command in the foreground with an explicit timeout; cite the output lines that carry a result,
  never a whole build log.

## Part 1, before anything else
State in your own words what this change does and what it is for, from the diff and the code it
touches. Follow the changed values to where they are produced and consumed. If you cannot tell what
some part is for, say so; "I cannot tell what this is for" is itself a finding.

## Part 2
Review it. For each finding: BLOCKING or non-blocking, a concrete failure scenario, and the command
output that demonstrates it. Verify by execution rather than reading where you can. Look at what the
change does not do as well as what it does, and at whether comments and allowlist text make claims
the code supports; where the change adds allowlist entries, check that each names something that
actually enforces what it claims.

Report Part 1 then Part 2, `shasum <DIFF_PATH>`, then a one-line verdict.
