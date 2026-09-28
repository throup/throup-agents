---
name: write-ticket
description: Write or rewrite a ticket in the project's tracker (YouTrack, Jira, GitHub Issues or another) so a product owner can read the title, the first two paragraphs and the acceptance criteria, and an implementer finds everything else in one collapsed block. Use when filing a ticket, reshaping one, merging duplicates, or folding an amendment back into a description.
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Write a ticket

One ticket serves two readers who never need each other's material. A product owner reads the
title, two paragraphs and the acceptance criteria and can prioritise from those alone. An
implementer opens one collapsed block and finds what the session established, so nothing is
re-derived. Each sentence goes to the reader who acts on it; a sentence neither acts on is cut.

The tracker is whichever the user's or the project's guidelines name, and its CLI or API is how
the ticket is read and written. Where the guidelines name none, ask before writing. Field names,
link types and state names below are described by role; read the tracker's own names from its
schema or its documentation rather than assuming them.

## Where a sentence belongs

- **Visible to the product owner** when it says what changes, whether anything is broken today,
  or what done looks like. Plain words: no file, script, step or variable names, no ticket keys,
  commits or run IDs, no tally of the items the ticket covers. A figure stays when the product
  owner prioritises on it; a fact the product owner would not act on is not visible. The
  strongest evidence leads, ahead of the mechanism that produces it. A table is visible only when
  the table is the scope and the task cannot be judged without it.
- **An acceptance criterion** when it is an outcome, or a constraint the implementer must honour.
  Constraints are stated plainly inside the AC because the ACs travel without the rest of the
  ticket. An AC that changes existing behaviour carries the code's own reason for that
  behaviour, read at the locator; where none is found, the block says so.
- **In the collapsed block** when the implementer would otherwise re-derive it: mechanism,
  locators pinned as "line N on `<base>` at `<sha>`", evidence, options not chosen, related
  tickets with one clause each. Provenance names the mechanism and measurement, not a change
  request, key, branch or person. A locator that no longer resolves is named here as gone, not
  dropped. Include what this ticket's implementer needs and no paragraph written for
  completeness. The block ends by naming the repository the change lands in, as a link and the
  files concerned, so tooling that starts from the ticket can find the code without reading the
  prose.
- **In the field, not the prose**: priority, and anything else the tracker holds as a field
  (assignee, component, labels). Where the tracker has no field for it, the guidelines say
  which label or line stands in.
- **Nowhere**: the argument for the priority, the narrative of how the finding was made, and
  anything the implementer can read from the repo.

## Shape, in this order

1. **Title.** An imperative action, at most twelve words.
2. **First paragraph, two sentences.** What is wrong or needed, and why it matters. A defect
   that has already shipped is sentence two.
3. **Second paragraph.** Current impact. Say "Nothing is broken today" when that is true, and
   say when the defect is not yet reproduced.
4. **`## Acceptance criteria`.** Numbered. Outcome ACs first; any closing constraint AC last.
   Dropping the heading is a convention change to say out loud, not a style call.
5. **One collapsed block**, `<details><summary>Technical detail for the implementer</summary>`
   where the tracker renders HTML in Markdown, else the tracker's own collapsible construct;
   where it has none, a `## Technical detail for the implementer` heading last. One bold lead-in
   per paragraph. Nothing sits between the ACs and the block.
6. **Attribution line.** `🤖 Generated with [Claude Code](https://claude.com/claude-code)`,
   or the line the user's guidelines name instead.

## Changing a ticket that exists

- Only tickets the user's account reported are edited; others are read for calibration, and a
  change to one is proposed as a comment or asked about first. A resolved ticket is not
  rewritten.
- A comment that changes scope changes the description in the same sitting: the ACs, a "folded
  in from the comment of `<date>`" line in the block, and the visible impact paragraph when a
  question is resolved or a blocker cleared.
- Merging duplicates: the survivor takes both AC sets and a "Merged from" paragraph in its
  block; the closed one is linked to the survivor as its duplicate (the tracker's duplicate link
  type, or a "Duplicate of" line where it has none), moved to the tracker's not-doing state as
  read from its schema, and given a one-paragraph comment pointing at the survivor.
- An umbrella's visible text names the shared problem; its block lists the subtasks by key with
  one clause each and repeats nothing from them.
- A search for existing tickets made before filing is validated against a ticket known to match
  first: a query with bad syntax returns nothing and reads exactly like an empty board.
- Re-read after writing. The tracker's update response under-reports what it applied.

A revision this skill's use suggests goes through the channel the user's guidelines name for
skill feedback, else to the user as one line in the outcome message. Nothing is applied to this
file during a run.
