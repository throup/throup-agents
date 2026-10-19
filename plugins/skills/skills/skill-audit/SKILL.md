---
name: skill-audit
description: Cut one skill to the sentences its confirmed purpose needs and no authority already states, then have a fresh blind reviewer judge the cut against that purpose and a baseline; for a review asked of a skill's change request, or an audit of a skill whose change request is open where the operator chooses it, post the results on it as one review. Use when a skill has grown, when a skill's change request is open or a review of one is asked for, when its text and the user's or project's guidelines or memory disagree, or when asked to audit, trim or shorten a skill. Not for processing suggestions already recorded for a skill, not for writing a skill from nothing, and not for a rule file or a prompt.
disable-model-invocation: false
allowed-tools: Agent, SendMessage, Bash, Read, Write, Edit, Grep, Glob
---

# Skill audit

An audit cuts one skill to the sentences its confirmed purpose needs and no authority already
states, and has a reviewer with none of the audit's context judge the cut against that purpose and
the baseline; a review makes no cut, and has that reviewer judge an open change request to a skill.
It produces the shorter skill, edited in place or through a change request, or on the review route
one review posted on the open change request, with an audit's cut as suggested changes and the
confirmed findings as comments; and a report: each reviewer's reading against the purpose, each
finding with what was done with it, and the conflicts, disputed findings and fixes outside the
skill's files left to the operator. The operator confirms the purpose before anything is cut, rules
on what the report escalates, confirms the post on the review route, and merges the change request
where the audit opened one. The audit changes no authority and adds no rule the baseline did not
state, other than a suggestion already recorded for the skill that the process the guidelines name
applies within the audit; a rule the skill has never had is a suggestion for the skill.

## Authorities

The texts in context wherever the skill loads. For a skill the operator confirms is loaded on other
machines: the skills its text points to, and those files of the skill's repo that are in context
wherever the skill loads, of which a plugin's skill has none. The user's guidelines and memory are
not authorities for it; they still name the channels, locations and definitions the audit itself
uses. For any other skill: the user's guidelines (`~/.claude/CLAUDE.md`); for a skill inside a repo,
that repo's `CLAUDE.md`, `AGENTS.md` and rules; any memory directory the harness loads, whose index
is read before anything else. Every step that names an authority uses this set. Name each file and
directory as the population, and say which files were read.

## Purpose first

Establish the route first, as Cut's first two paragraphs do. Read the frontmatter description, the
opening paragraph and the artefacts the skill has produced: any suggestions recorded for it in the
channel the guidelines name for skill feedback, and the tickets, files or notes it wrote; on the
review route, read the description and the opening in its worktree. Read no further into the body.
Where this session already has the whole skill loaded (an audit of `skill-audit` itself, or of a
skill the session was started with), reading no further cannot hold: tell the operator so before
proposing the purpose, and the operator chooses to carry on or to start the audit in a fresh
session. Write the purpose in two or three sentences: what the skill produces, who acts on its
output, what it must not become. Propose with it whether the skill is loaded on other machines, from
`if [ -d "<dir>" ]; then git -C "<dir>" ls-files --error-unmatch . >/dev/null 2>&1 && git -C "<dir>" remote | grep -q . && echo yes || echo no; else echo "no such directory"; fi`,
where `<dir>` is the absolute path of the directory the skill loads from, or for a plugin cache the
skill's directory in a checkout of its plugin's source repo, or on the review route its directory in
the review route's worktree; where it prints `no such directory`, correct the path before proposing,
and where it cannot be corrected, stop and say so. Put both to the operator and stop until both are
confirmed; the confirmed purpose, as Apply rewrites it, is the specification for every later step.

## Cut

Where the operator asks for a review of a change request to a skill, the route is the review route.
Otherwise, establish the route from the directory the skill loads from, symlinks followed:
`if d=$(cd "<that directory>" && pwd -P); then git -C "$d" ls-files --error-unmatch . >/dev/null 2>&1 && git -C "$d" rev-parse --show-toplevel || echo untracked; else echo "no such directory"; fi`.
A skill loaded from a plugin cache (`~/.claude/plugins/cache/`) is audited in a checkout of its
plugin's source repo, as a tracked skill; where there is none, stop and say so. For a tracked skill,
list the change requests open on its repo that touch its directory, with the code host the
guidelines name, asking where they name none; on GitHub, in the skill's directory:
`gh pr list --state open --limit 100 --json number,files --jq ".[] | select(any(.files[]; .path | startswith(\"$(git rev-parse --show-prefix)\"))) | .number"`,
which matches every change request where the skill is at the repo's root; there, ask the operator
which change request, if any, is the skill's. Where one is open, put it to the operator, who chooses
the review route on it or the tracked route, and with several names the one the review route takes.

The review route has two entries: a review, which the operator asks for, makes no cut and edits
nothing; an audit of a skill whose change request is open makes the cut. On either, fetch the change
request's head commit from the code host and record its SHA (on GitHub,
`git fetch <remote of the change request's repo> pull/<number>/head`), take as
its base commit the merge base of its head and its base branch as the code host has it, fetched, and
make a worktree detached at the head, not in the checkout a live skill loads from; every step that
reads or edits the skill at the head uses that worktree. Open no ticket and no change request, and
commit nothing. On a review, the skill's text at the head is the cut the reviewer judges, its text
at the base is the baseline, none for a skill the change request adds, and the rest of Cut is
skipped. On an audit, the cut is made in the worktree and left uncommitted; the head is the
baseline.

Off the review route, an untracked skill is edited in place; back its files up first to a
directory that does not exist yet, outside any skills directory:
`mkdir -p "<parent>" && mktemp -d "<parent>/<yyyy-mm-dd>-XXXXXX"`, where `<parent>` is the location
the guidelines name, as an absolute path, or else `$HOME/.claude/skill-backups/<skill>`, and a
tracked skill is a ticket, a branch and a change request in the repo the command printed, using
the tracker and code host the guidelines name, and asking before writing where they name none; the
edit is made in a worktree, not the checkout a live skill loads from, and the base commit is the
baseline.

Read the body against the purpose. Cut a sentence an authority states and a sentence the purpose
does not need. Where a reason being cut is the only statement of an action, keep the action as an
instruction. Agent-loaded text is imperative and terse: current facts, no motivation, no worked
examples, no history and no prediction, unless the authorities set another register. Write or
rewrite the opening as one paragraph stating the confirmed purpose: what the skill produces, who
acts on it, and what it must not become. Before stage one is written, read the opening against each
rule in the body and the frontmatter description, and reword the opening until it contradicts
neither; where the contradiction is with the confirmed purpose, report it to the operator instead.
The frontmatter is otherwise unchanged, except that a statement of when the skill is not invoked,
cut from the body, lands in the description's "Not for" sentence. The cut adds nothing a recorded
suggestion proposes.

## Blind review

Write the brief in two stages. Stage one carries the cut, the authorities as a population, a command
that checks each file the job names, and a place for the findings; it asks the reviewer only to
write, from the cut alone, what the skill produces, who acts on its output and what it must not
become, under `## Reading` in that place, and under `## Loaded context` the primary working
directory, branch name and commit subjects its session loaded before its first tool call, and to
return with no findings and the job not closed, its second stage following by message. Write stage
two only once both are recorded, to a file in a new temporary directory that also holds stage two's
inputs: the confirmed purpose as the specification, the baseline, for a review the change request's
diff of the skill's files, the skill's artefacts, each named with the version of the skill that
produced it, and a request to give each finding a kind and a cited line. That version is the one the
operator names as having run, where the operator has named one, else on the tracked and review
routes the skill's last commit before the artefact was made, from this command run in the skill's
repo:
`git log -1 --format=%h --before="<the artefact's creation time, with time of day and zone>" -- "<skill directory>"`;
it is unknown where that fails or prints nothing, where the artefact is dated only to the day, and
on the untracked route. The kinds:

1. a rule dropped that the purpose needs and no authority states;
2. a sentence kept that the purpose does not need or an authority states;
3. a sentence that reads two ways, or whose meaning changed where the baseline's reading was
   right;
4. a rule the skill's own artefacts do not follow, or a structure they have that no rule asks for.

Withheld from the reviewer: editing anything but the reading, the loaded context, the findings and,
on the handover route, the symlink the handover's close removes; backups other than the baseline;
before stage two, the purpose as confirmed, the baseline and the artefacts, backups and git history
included; the audit's reasoning, and on the tracked and review routes the ticket, the change request
and the commit messages that carry it; invoking the skill under review. Stage one's withheld list
allows, once stage two arrives, the stage-two file and the directory holding stage two's inputs.
Neither stage, nor any message to the reviewer, carries more than this section and the Apply section
list. Launch with the Agent tool, subagent type `general-purpose`, never a fork of this session. On
the tracked and review routes, before each launch, run, with `<dir>` this session's primary working
directory, and `<branch>`, `<key>`, `<repo>` and `<range>` the audit branch, the ticket key, the
audit's worktree and `<baseline commit>..HEAD`, or on the review route the change request's head
branch, its ticket key if any, the review route's worktree and `<base commit>..<head commit>`,
`{ echo "<dir>"; git -C "<dir>" branch --show-current; git -C "<dir>" log -5 --format=%s; } | grep -F -f <({ printf '%s\n' "<branch>" "<key>"; git -C "<repo>" log --format=%s "<range>"; } | grep .)`;
each line it prints is that reviewer's exposure. The check has run only where its `git log`, run
alone, exits 0; where it does not, fix `<range>` and run the check again. Send stage two to that
same agent with SendMessage, with this and nothing more:

```
Read <absolute path of stage two> and continue.
```

Where the reviewer returns without a reading or a loaded context, closes the job before stage two or
does not continue with it, launch a new reviewer from a newly written stage one, on the handover
route a new handover; where that one fails the same way, or SendMessage is not available, stop and
say so.

Where the `handover` skill is installed (as `handover` or `delivery:handover`), is not the skill
under audit, and the session is in a git checkout for the pointer, the brief is a handover written
with it: stage one is the handover, with Verify lines for the checks, `## Reading` and
`## Loaded context` appended to the store file, findings returned as its `## Closed` note once
stage two is done, and the store's other files withheld; stage two is outside the store. Place the
symlink and launch with this and nothing more:

```
You have been pointed at <absolute path of the symlink>. Invoke the handover skill and receive it.
```

Once the audit is done with that reviewer, whether it closed the handover, was replaced or the audit
stopped, remove that reviewer's symlink if it is still in place.

Where `handover` is not used, stage one is a file in a new temporary directory, with the reading and
the findings to be appended under closing headings, and launch with this and nothing more:

```
Read <absolute path of the brief> and do the job it describes.
```

Where `handover` is not installed, the report recommends the `delivery` plugin from
[throup-agents](https://github.com/throup/throup-agents).

## Apply

A confirmed finding is applied to the skill, except on a review, which edits nothing. A fix made
after the blind review changes only the sentence its finding cites, or for a dropped rule restores
only the baseline sentence it cites. A rewording that would change what a rule applies to goes to
the operator as a choice, as does a finding that offers keeping a reason being cut or cutting the
rule it supported; each is made only as the operator rules. A ruling that changes or narrows what
the skill is for also rewrites the confirmed purpose and, where there is a cut, the skill's opening
paragraph to state it, in the same step, reading that opening against the body and the description
as Cut does, and the rewritten purpose goes to the operator to confirm before the next reviewer is
launched. Every later reviewer's stage two states each such ruling, without the audit's reasoning,
and carries the purpose as last confirmed. A finding whose fix lands outside the skill's files, a
conflict between the skill and an authority, and a finding the auditor disputes are reported to the
operator with the line, and nothing else is edited. For a skill loaded on other machines, read the
user's guidelines and memory against the skill, and report each conflict with them the same way,
proposed as the guidelines overriding the skill's default locally; where the operator rejects that
framing, the operator chooses to keep the skill's sentence, reword it to match the guidelines, or
cut it. Suggestions already recorded for the audited skill are decided against the new text by the
process the guidelines name for skill feedback, after the audit unless the guidelines have it run
within the audit; where they name none, the report lists them. The reviewer's suggestion for
`handover` goes through that channel, else into the report.

Keep the text the reviewer judged: on the tracked route its commit, otherwise a copy in a new
directory made as the backup's is. Every edit made after it, fixes and suggestions applied within
the audit alike, goes to one new reviewer launched as in Blind review on the current text, whose
stage two adds the diff from the judged text to the current text and asks only for findings the diff
causes, on any line. Its findings are reported to the operator with the line, and nothing more is
edited; an edit the operator then asks for is reviewed the same way. On the tracked route, the audit
then ends when its change request is ready for human review as the guidelines define it, else as
`handover` defines it where installed, else when every relevant commit is pushed, a fresh
review at the head found nothing blocking, and CI is green at the head.

On the review route, once the operator has ruled on what the report escalates, put to the operator
one comment-only review of the change request against the recorded head SHA (on GitHub, event
`COMMENT` with that `commit_id`): on an audit, the diff from the baseline to the current text as
suggested changes, each on its line where that line is in the change request's diff; each confirmed
finding that diff does not carry as a comment on the head's line its cited line corresponds to,
where that line is in the diff; and everything else in the review's body, quoting its line with the
replacement or the finding. Post it once the operator confirms the post; the audit then ends.

Report each reviewer's reading beside the purpose it judged against, as two columns of one table;
each reviewer's exposure: the check's lines, and each line of its `## Loaded context` that carries
the check's `<branch>`, its `<key>` or a commit subject from its `<range>`; word counts before and
after; each finding's kind with applied or reported; the conflicts awaiting the operator; and on the
review route the posted review's link.

A revision to this skill that the run suggests goes through the channel the user's guidelines name
for skill feedback, else to the user as one line in the report with the evidence from the run.
Where there is none, the report says so.
