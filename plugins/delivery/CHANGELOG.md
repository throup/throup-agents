# Changelog

All notable changes to the `delivery` plugin. Versions follow [semver](https://semver.org/).

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
