---
name: skill-audit
description: Cut one skill to the sentences its confirmed purpose needs and no authority already states, then have a fresh blind reviewer judge the cut against that purpose and a baseline. Use when a skill has grown, when its text and the user's or project's guidelines or memory disagree, or when asked to audit, trim or shorten a skill. Not for processing suggestions already recorded for a skill, not for writing a skill from nothing, and not for a rule file or a prompt.
disable-model-invocation: false
allowed-tools: Agent, SendMessage, Bash, Read, Write, Edit, Grep, Glob
---

# Skill audit

An audit cuts one skill to the sentences its confirmed purpose needs and no authority already
states, and has a reviewer with none of the audit's context judge the cut against that purpose and
the baseline. It produces the shorter skill, edited in place or through a change request, and a
report of what it did not apply: disputed findings, fixes outside the skill's files and conflicts
with an authority. The operator confirms the purpose before anything is cut, decides the report,
and merges the change request where there is one. The audit changes no authority and adds no rule
the baseline did not state; a rule the skill has never had is a suggestion for the skill.

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

Read the frontmatter description, the opening paragraph and the artefacts the skill has produced:
any suggestions recorded for it in the channel the guidelines name for skill feedback, and the
tickets, files or notes it wrote. Read no further into the body. Write the purpose in two or three
sentences: what the skill is for, who acts on its output, what it must not become. Propose with it
whether the skill is loaded on other machines: yes where the skill is tracked in a repo that has a
remote, else no, from
`git -C "<dir>" ls-files --error-unmatch . >/dev/null 2>&1 && git -C "<dir>" remote | grep -q . && echo yes || echo no`,
where `<dir>` is the absolute path of the directory the skill loads from, or for a plugin cache the
skill's directory in a checkout of its plugin's source repo. Put both to the operator and stop until
both are confirmed; the confirmed purpose, as Apply rewrites it, is the specification for every
later step.

## Cut

Establish the route from the directory the skill loads from, symlinks followed:
`if d=$(cd "<that directory>" && pwd -P); then git -C "$d" ls-files --error-unmatch . >/dev/null 2>&1 && git -C "$d" rev-parse --show-toplevel || echo untracked; else echo "no such directory"; fi`.
A skill loaded from a plugin cache (`~/.claude/plugins/cache/`) is audited in a checkout of its
plugin's source repo, as a tracked skill; where there is none, stop and say so. An untracked skill
is edited in place; back its files up first to a directory that does not exist yet, outside any
skills directory: `mkdir -p "<parent>" && mktemp -d "<parent>/<yyyy-mm-dd>-XXXXXX"`, where
`<parent>` is the location the guidelines name, as an absolute path, or else
`$HOME/.claude/skill-backups/<skill>`. A tracked skill is a ticket, a branch and a change request in
the repo the command printed, using the tracker and code host the guidelines name, and asking before
writing where they name none; the edit is made in a worktree, not the checkout a live skill loads
from, and the base commit is the baseline.

Read the body against the purpose. Cut a sentence an authority states and a sentence the purpose
does not need. Where a reason being cut is the only statement of an action, keep the action as an
instruction. Agent-loaded text is imperative and terse: current facts, no motivation, no worked
examples, no history and no prediction, unless the authorities set another register. Write or
rewrite the opening as one paragraph stating the confirmed purpose: what the skill produces, who
acts on it, and what it leaves to them. The frontmatter is otherwise unchanged, except that a
statement of when the skill is not invoked, cut from the body, lands in the description's "Not for"
sentence. The cut adds nothing a recorded suggestion proposes.

## Blind review

Write the brief in two stages. Stage one carries the cut, the authorities as a population, a command
that checks each file the job names, and a place for the findings; it asks the reviewer only to
write what the skill is for, from the cut alone, under `## Reading` in that place, and under
`## Loaded context` the primary working directory, branch name and commit subjects its session
loaded before its first tool call, and to return with no findings and the job not closed, its second
stage following by message. Write stage two only once both are recorded, to a file in a new
temporary directory: the confirmed purpose as the specification, the baseline, the skill's
artefacts, each named with the version of the skill that produced it, and a request to give each
finding a kind and a cited line. That version is the one the operator names as having run, where
the operator has named one, else on the tracked route the skill's last commit before the artefact
was made, from this command run in the skill's repo:
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
included; the audit's reasoning, and on the tracked route the ticket, the change request and the
commit messages that carry it; invoking the skill under review. Neither stage, nor any message to
the reviewer, carries more than this section and the Apply section list. Launch with the Agent tool,
subagent type `general-purpose`, never a fork of this session; the reviewer loads this session's
primary working directory, its branch name and its recent commit subjects. On the tracked route,
before each launch, run, with `<dir>` that primary working directory and `<worktree>` the audit's
worktree,
`{ echo "<dir>"; git -C "<dir>" branch --show-current; git -C "<dir>" log -5 --format=%s; } | grep -F -f <({ printf '%s\n' "<audit branch>" "<ticket key>"; git -C "<worktree>" log --format=%s "<baseline commit>..HEAD"; } | grep .)`;
each line it prints is that reviewer's exposure. Send stage two to that same agent with SendMessage,
with this and nothing more:

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

A confirmed finding is applied to the skill. A fix made after the blind review changes only the
sentence its finding cites, or for a dropped rule restores only the baseline sentence it cites. A
rewording that would change what a rule applies to goes to the operator as a choice, as does a
finding that offers keeping a reason being cut or cutting the rule it supported; each is made only
as the operator rules. A ruling that changes or narrows what the skill is for also rewrites the
confirmed purpose and the skill's opening paragraph to state it, in the same step, and the rewritten
purpose goes to the operator to confirm before the next reviewer is launched. Every later reviewer's
stage two states each such ruling, without the audit's reasoning, and carries the purpose as last
confirmed. A finding whose fix lands outside the skill's files, a conflict between the skill
and an authority, and a finding the auditor disputes are reported to the operator with the line, and
nothing else is edited. For a skill loaded on other machines, read the user's guidelines and memory
against the skill, and report each conflict with them the same way, proposed as the guidelines
overriding the skill's default locally; where the operator rejects that framing, the operator
chooses to keep the skill's sentence, reword it to match the guidelines, or cut it. Suggestions
already recorded for the audited skill are decided against the new text by the process the
guidelines name for skill feedback; where they name none, the report lists them. The reviewer's
suggestion for `handover` goes through that channel, else into the report.

Keep the text the reviewer judged: on the tracked route its commit, on the untracked route a copy in
a new directory made as the backup's is. Every edit made after it, fixes and applied suggestions
alike, goes to one new reviewer launched as in Blind review on the current text, whose stage two
adds the diff from the judged text to the current text and asks only for findings the diff causes,
on any line. Its findings are reported to the operator with the line, and nothing more is edited; an
edit the operator then asks for is reviewed the same way. A tracked skill's audit then ends when its
change request is ready for human review as the guidelines define it, else as `handover` defines it
where installed; otherwise when every relevant commit is pushed, a fresh review at the head found
nothing blocking, and CI is green at the head.

Report each reviewer's reading beside the purpose it judged against, as two columns of one table;
each reviewer's exposure: the check's lines, and each line of its `## Loaded context` that carries
the audit branch, the ticket key or an audit commit subject; word counts before and after; each
finding's kind with applied or reported; and the conflicts awaiting the operator.

A revision to this skill that the run suggests goes through the channel the user's guidelines name
for skill feedback, else to the user as one line in the report with the evidence from the run.
Where there is none, the report says so.
