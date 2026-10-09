---
name: skill-audit
description: Cut one skill to the sentences its confirmed purpose needs and no authority already states, then have a fresh blind reviewer judge the cut against that purpose and a baseline. Use when a skill has grown, when its text and the user's or project's guidelines or memory disagree, or when asked to audit, trim or shorten a skill. Not for processing suggestions already recorded for a skill, not for writing a skill from nothing, and not for a rule file or a prompt.
disable-model-invocation: false
allowed-tools: Agent, Bash, Read, Write, Edit, Grep, Glob
---

# Skill audit

A skill an agent loads says only what its job needs and nothing an authority already says; every
other sentence costs context on each action, and a rule stated twice drifts. An audit leaves a
shorter skill and a list of the places it disagrees with the rules it sits under. The operator
acts on both: they say what the skill is for before a word is cut, decide any finding the auditor
disputes, and settle each conflict with an authority. The audit edits the skill and reports
everything else; it changes no authority and adds no rule the baseline did not state. A rule the
skill has never had is a suggestion for the skill, not part of the cut.

## Authorities

The texts in context when the skill loads: the user's guidelines (`~/.claude/CLAUDE.md`); for a
skill inside a repo, that repo's `CLAUDE.md`, `AGENTS.md` and rules; any memory directory the
harness loads, whose index is read before anything else. Name each file and directory as the
population, and say which files were read; a reviewer bounded to a subset cannot tell "not
stated" from "stated in a file it was not given".

## Purpose first

Read the frontmatter description, the opening paragraph and the artefacts the skill has produced:
any suggestions recorded for it in the channel the guidelines name for skill feedback, and the
tickets, files or notes it wrote. Read no further into the body. Write the purpose in two or three
sentences: what the skill is for, who acts on its output, what it must not become. Put it to the
operator and stop until it is confirmed; the confirmed text is the specification for every later
step.

## Cut

Establish the route from the directory the skill loads from, symlinks followed:
`if d=$(cd "<that directory>" && pwd -P); then git -C "$d" ls-files --error-unmatch . >/dev/null 2>&1 && git -C "$d" rev-parse --show-toplevel || echo untracked; else echo "no such directory"; fi`.
A skill loaded from a plugin cache (`~/.claude/plugins/cache/`) is audited in a checkout of its
plugin's source repo, as a tracked skill; where there is none, stop and say so. An untracked skill
is edited in place; back its files up first, to the location the guidelines name or else
`~/.claude/skill-backups/<skill>/<yyyy-mm-dd>/`, outside any skills directory so the copy does not
register. A tracked skill is a ticket, a branch and a change request in the repo the command
printed, using the tracker and code host the guidelines name, and asking before writing where they
name none; the edit is made in a worktree, not the checkout a live skill loads from, and the base
commit is the baseline.

Read the body against the purpose. Cut a sentence an authority states and a sentence the purpose
does not need. Agent-loaded text is imperative and terse: current facts, no motivation, no worked
examples, no history and no prediction, unless the authorities set another register. Write or
rewrite the opening as one paragraph stating the confirmed purpose: what the skill produces, who
acts on it, and what it leaves to them. The frontmatter is otherwise unchanged, except that a
statement of when the skill is not invoked, cut from the body, lands in the description's "Not
for" sentence. The cut adds nothing a recorded suggestion proposes; those are decided after the
review, against the new text.

## Blind review

Write a brief carrying the confirmed purpose as the specification, the baseline, the skill's
artefacts, the authorities as a population, a command that checks each file the job names, and a
place for the findings. It asks the reviewer to state first what the skill is for from the cut
alone, and to give each finding a kind and a cited line:

1. a rule dropped that the purpose needs and no authority states;
2. a sentence kept that the purpose does not need or an authority states;
3. a sentence that reads two ways, or whose meaning changed where the baseline's reading was
   right;
4. a rule the skill's own artefacts do not follow, or a structure they have that no rule asks for.

Withheld from the reviewer: editing anything; backups other than the baseline; the audit's
reasoning, and on the tracked route the ticket, the change request and the commit messages that
carry it; invoking the skill under review. Launch with the Agent tool, subagent type
`general-purpose`, never a fork of this session, so the reviewer starts with none of the audit's
context.

Where the `handover` skill is installed (as `handover` or `delivery:handover`) and is not the
skill under audit, the brief is a handover written with it: Verify lines for the checks, findings
returned as its `## Closed` note, and the store's other files withheld. Place the symlink and
launch with this and nothing more:

```
You have been pointed at <absolute path of the symlink>. Invoke the handover skill and receive it.
```

Otherwise write the brief to a file in a new temporary directory, with the findings to be appended
under a closing heading, and launch with this and nothing more:

```
Read <absolute path of the brief> and do the job it describes.
```

Where `handover` is not installed, the report recommends the `delivery` plugin from
[throup-agents](https://github.com/throup/throup-agents).

## Apply

A confirmed finding is applied to the skill. A finding whose fix lands outside the skill's files,
a conflict between the skill and an authority, and a finding the auditor disputes are reported to
the operator with the line, and nothing else is edited. Suggestions already recorded for the
audited skill are decided against the new text by the process the guidelines name for skill
feedback; where they name none, the report lists them. The reviewer's suggestion for `handover`
goes through that channel, else into the report. A tracked skill's audit ends when its change
request is ready for human review as the guidelines define it, else as `handover` defines it where
installed; otherwise when every relevant commit is pushed, a fresh review at the head found nothing
blocking, and CI is green at the head.

Report word counts before and after, each finding's kind with applied or reported, and the
conflicts awaiting the operator.

A revision to this skill that the run suggests goes through the channel the user's guidelines name
for skill feedback, else to the user as one line in the report with the evidence from the run.
Where there is none, the report says so.
