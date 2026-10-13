---
name: staged-delivery
description: "Plan and deliver a software change in blocks with native implementers that self-validate and iterate, parallel ownership of separate files, and an independent validator plus reviewer gate at strategic milestones—normally only at completion. Use when the owner asks to implement a development plan with the staged-delivery workflow."
---

# Staged delivery

Start with a concrete plan. Keep implementation moving through small, testable
blocks; use independent functional validation and code review at the milestones
that justify them. **Default: one final gate, not one gate per block.**

The coordinator (the main session running this skill) is architect and
supervisor. It resolves implementation questions, reviews evidence, assigns
ownership and decides which findings need correction. Implementers own delivery
and self-validation. A separate validator and reviewer approve each planned gate.
Use native subagents with the dedicated roles below.

## Plan before implementation

Read the project's instruction files (`CLAUDE.md`, `AGENTS.md` and nested ones
for the subtrees you will touch), current code and canonical product docs. Use
an existing approved plan when available; revise only what new evidence changes.
Save the execution plan at the owner's requested path, otherwise under
`<worktree>/tmp/<task>/PLAN.md` (make sure `tmp/` is gitignored). Keep it concise
and usable by a fresh agent:

- Outcome, observable acceptance scenarios, constraints and explicit exclusions.
- Blocks with deliverables, dependencies, owned paths, entry points to exercise
  and completion evidence. Identify which can run concurrently.
- Gate placement, scope and reason for any gate before completion.
- Real dependencies, test data, process ownership, evidence directory and checks.
- Commit, publication and merge authority granted by the owner.

Record the task's starting revision and pre-existing dirty/untracked work so
later review can distinguish task changes from unrelated edits.

For a plan-only request, stop at the document. When implementation is already
authorized, continue from the plan without adding an approval ceremony. Routine
design decisions belong to the coordinator; ask the owner only for material
missing intent, authority or a decision outside the agreed scope.

### Blocks and gates

A **block** is a bounded implementation outcome that its implementer can validate.
A **stage** is the set of blocks up to the next independent gate. One stage may
contain the entire task. Block completion requires self-validation and the
coordinator's evidence check; it does not imply independent approval.

Add an earlier gate only when failure would invalidate substantial dependent work
or expose a critical contract, such as durable recovery, isolation, data
migration, security or an external execution boundary. Write that reason and the
dependent blocks it holds back in the plan. A directory boundary, a new module or
the number of blocks alone does not justify a gate. Reassess placement if
observed risk changes; explain the change instead of silently reviewing every
block.

Use CLI/API-first development when it proves the actual behavior cheaply, then
adapt the GUI to the working contracts. CLI and GUI can share one final gate;
separate gates are appropriate when the plan justifies the CLI foundation as
critical. Visual-only work should validate through the browser from the start.

## Roles and models

| Role | Agent | Model |
| --- | --- | --- |
| Coordinator | main session | any (Opus 5.5 medium or high recommended) |
| Implementer | `staged-implementer` | Sonnet 5.5, high effort |
| Validator | `staged-validator` | Sonnet 5.5, high effort |
| Reviewer | `staged-reviewer` | Opus 5.5, medium effort |

The roles pin their own model and effort in their agent definitions; do not
inherit them from the coordinator. If the runtime lets you pass `model`/`effort`
on the subagent call, pass the values above explicitly. Report an unavailable
model instead of silently substituting one. Independent roles must run in
separate sessions even if their models match. Name `staged-delivery` in each
brief to identify the workflow.

Give each agent a short brief with:

- Absolute worktree, branch, plan path, stage/block IDs and assigned goal.
- Owned write paths, exclusions, dependencies, agreed interfaces and other writers.
- Acceptance scenarios, checks and actual CLI/API/browser/worker entry points;
  distinguish real services from controlled fixtures or fault injection.
- Absolute gitignored evidence directory under `<worktree>/tmp/<task>/<run>/`,
  safe credential loading, isolated test data, runtime URL/process owner,
  lifecycle authority, bounded waits and cleanup obligations.
- Self-validation evidence or gate evidence as applicable, original diff baseline,
  next allowed block or gate stop, and commit/publication authority.

Load project or domain skills only as needed, and reuse the project's existing
test drivers and QA helpers instead of writing new ones. Do not require a UI or
external service for a component that does not use one.

## Parallel implementation and integration

The coordinator assigns **one writer per file at a time**. Parallelize independent
blocks with non-overlapping write paths and usable interfaces. Tell every writer
that others share the checkout: preserve their changes and stay inside ownership.
Disjoint files alone are insufficient if the contracts or runtime state conflict.

Shared contracts, registries, manifests, lockfiles, schemas and other common
files get one named owner. Establish a needed contract first; let dependents
proceed after its self-validation unless the plan puts a gate there. Serialize
shared-file changes or reassign ownership explicitly. Dependency installs,
formatting across scopes, builds and shared service reconfiguration also need
coordination.

Each implementer may advance through the ordered blocks already assigned for the
stage after validating the current one and reporting evidence. For a new or
changed assignment, the coordinator checks dependencies and grants the next scope.
Assign one implementer to exercise the combined path after parallel work lands.
Passing isolated checks does not prove integration. No block crosses its planned
gate until both independent verdicts are accepted.

## Implementer self-validation loop

For each block: reproduce or add a meaningful failing behavior check when
appropriate, make the smallest coherent change, run focused checks, and exercise
the actual entry point whenever there is behavior to test. Iterate until the
block's acceptance scenarios pass. Pure documentation changes need suitable
document/config validation, not invented live tests.

Use the same implementer for related blocks within its assigned stage and for
corrections; avoid handing its own validation to another agent. Report paths,
observed outcomes, commands, evidence and limitations as each block completes.
When a foundation has no live entry point yet, record exactly what is covered
and which later integration block will prove it. Before a gate, implemented
runtime behavior must have real self-validation; unit tests alone are insufficient.

The coordinator resolves doubts and distinguishes product failures, test-driver
mistakes and environment failures. Keep iteration focused; do not weaken goals,
raise limits or substitute an easier path merely to obtain a pass. If iterations
stop producing new evidence, diagnose and revise the approach. Recover or escalate
a real blocker rather than loop indefinitely or mark a blocked flow approved.

Retain a healthy session while it provides useful continuity. Start a fresh
implementer after a stage handoff, crash, stale/confused report or unproductive
context growth; hand over the concise brief and evidence. A block number change
alone does not require replacement. Where sessions cannot resume, continue the
logical assignment through a fresh invocation and explicit handoff; do not claim
the previous session's memory survived.

Run relevant focused checks after edits, then integrated package checks at the
gate. Reuse valid results for unchanged code; do not run every test suite after
every block or correction. Required repository and publication checks still
apply. Keep observed behavior separate from quality claims.

## Independent strategic gate

Once all gate-scope blocks are self-validated and integrated, freeze their source
and record its identity (commit plus dirty/untracked diff or source hashes).
Preserve the baseline from **before the first block covered by this gate**;
intermediate commits must not hide earlier changes from review.

Launch the independent pair, concurrently when they can share a stable source
and compatible runtime safely:

- **Validator (`staged-validator`):** independently exercise the assigned
  scenarios through real entry points, inspect outcomes and relevant persisted
  state/screenshots. Return PASS/FAIL/BLOCKED per scenario and evidence. It may
  write temporary QA artifacts only. Implementer results are input, not its verdict.
- **Reviewer (`staged-reviewer`):** read the cumulative gate diff, necessary
  surrounding contracts and evidence. Return concrete actionable findings with
  severity, path/line, failure and minimal correction, or clear. No writes, tests,
  extra review bots or publication. It may start from self-validation while
  independent QA runs; the gate closes only after both results apply to the same
  source.

The coordinator assesses findings and sends one coherent correction brief to the
owning implementer(s). Explain rejected findings using evidence; do not dismiss
an unresolved contract violation to force approval. Pause affected QA while its
source changes. The implementer fixes and self-validates; the independent roles
recheck affected or unproven scenarios/boundaries, preserving valid prior evidence
and the original baseline. Add broader checks only if the correction's impact
requires them. Repeat until functional QA passes and review has no unresolved
accepted findings. BLOCKED is not PASS.

After an intermediate gate, release dependent blocks. A final gate must prove
the overall user outcome and cover all changes not previously reviewed, including
later edits to accepted foundations and their integration. It need not re-review
unchanged foundations or replay every earlier passing scenario. Record exactly
which earlier evidence remains valid.

## Runtime, acceptance and delivery

The coordinator owns shared server lifecycle and delegates it explicitly. Keep
one dev server per checkout; do not build into its active output. Parallel
agents share a stable server with separate test identities, or use explicitly
isolated services/data. Never mix incompatible configurations against shared state.
Retain child handles and launch identity (worktree, PID, start/run ID, URL); use
bounded waits and clean up only owned processes. Do not trust a stale PID alone
or kill by broad process name/port — other checkouts may run identical commands.
Put temporary scripts, browser profiles and logs in the assigned evidence
directory, with child-scoped `TMPDIR` where needed. Keep credentials and real-user
data out of Git.

Specialists or owners mandated by the project's instructions (for example for
CI/CD, infrastructure or deployment) retain ownership; delegate to them instead
of absorbing those edits for convenience. Honor tool refusals and existing
authorization.

Commit accepted gate scope after inspecting the diff/status and running required
checks. If useful, explicitly authorize local checkpoint commits earlier; label
them as pending gate approval and retain the original review baseline. Do not
let checkpoint commits or passing block tests masquerade as independent acceptance.
If the accepted source is already committed, record gate acceptance against
those commits; no empty acceptance commit is needed.

Follow the owner's existing publication scope. A request to write a plan or use
this skill does not itself authorize push, PR, deployment or merge. When publication
is authorized and the final gate passes, delegate normal push/PR creation to an
implementer, checking the entire branch diff and existing PRs. Honor hooks;
publication failures require diagnosis and minimal fixes with affected validation.
Do not merge without explicit authority.

Close with the delivered outcome, relevant checks/limits, usable URLs or commands
and actual commit/PR status. If the work stays unfinished, update the handoff note
named in the plan (if any) and clear completed task details. No extra owner
intervention is needed for routine corrections or gates already authorized by
the plan.
