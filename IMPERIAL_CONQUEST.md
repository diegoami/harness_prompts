You are setting up the agent-driven build harness for PROJECT, before any product
work. The harness is: auto-loaded instructions, a process contract, a task
catalogue, GitHub as the only status store, and a /run-task skill that runs one
task end to end. Build the harness as a branch + PR and have it reviewed before
using it.

FILL THESE IN FIRST — everything below references them:
  PROJECT:            <name + one-line purpose>
  REPO:               <owner/name>
  STACK:              <language / toolchain>
  CHECK:              <build command>            (expect zero warnings)
  UNIT:               <unit-test command>
  FULL:               <full-suite command>
  N:                  3                          (full-suite repeats before big changes)
  WORKTREE_ROOT:      <e.g. C:\Users\<you>\projects\<project>-work\>
  IMPLEMENTER MODEL:  <model used for implementation>
  REVIEWER MODEL:     <a different model/family for review>
  LIVING REFERENCE:   <wiki/discussions, or "none, keep it in-repo">

REFERENCE IMPLEMENTATION (read before writing; replicate the pattern, not the
content): https://github.com/diegoami/imperial_conquest_2
  - docs/operating-guide.md      the entry point and the operating rules
  - docs/build-process.md        the process contract and prompt templates
  - docs/task-catalogue.md §1-3  graph, entries, index
  - docs/release-plan.md         versions gated by merged issues
  - .github/                     issue/PR templates, CODEOWNERS, CI

DELIVERABLES

1. CLAUDE.md — auto-loaded, short: only the rules that must never be forgotten,
   plus a pointer to the operating guide. Do not let it grow into a manual.

2. docs/operating-guide.md — where everything lives, how the project is operated,
   the standing user preferences, and the collaboration norms. This is the first
   document every session reads.

3. docs/build-process.md — the contract:
   - roles: main session (plans, dispatches, triages, merges) and two subagents,
     implementer and reviewer, the reviewer on a different model;
   - the task loop: dispatch implementer -> independent review -> rework (at most
     two rounds) -> merge only with an approving review and green CI -> apply the
     doc claims the merge made stale;
   - the five review gates: DoD independently reproduced; provenance of every
     number/claim; determinism; scope (changes only inside the task's Owns list);
     correctness sweep;
   - DoD lines are single runnable assertions and are not negotiable by the
     implementer;
   - bugs in merged code are filed and triaged, never patched from inside another
     task; non-blocking findings become one follow-up issue per merge;
   - git conventions: branch task/T<nn>-<slug>, commit "T<nn>: subject",
     squash-merge to main;
   - concurrency: one task in flight, the main checkout belongs to the main
     session, agents work in their own worktrees;
   - implementer and reviewer prompt templates, and the /run-task skill as a
     fenced block (the fenced text is the source of truth).

4. docs/task-catalogue.md — the plan:
   - a dependency graph (merge-after = hard, start-after = soft), waves, and the
     critical path;
   - one entry per task: Branch, Owns (the only paths the implementer may touch),
     Scope, Done-when (each line one runnable check), model/effort, reviewer, and
     dependencies;
   - an index linking every task to its GitHub issue. Issues are pointers to the
     entries, never copies.
   - Keep the graph and the totals generated or checked; prose counts drift.

5. docs/design.md and docs/design-audit.md — what is being built, and what is
   actually known. Tag every claim [confirmed] / [derived] / [designed] /
   [open] from day one, with a source for each.

6. docs/release-plan.md — SemVer; a small ladder of releases, each gated by a set
   of merged task issues tracked with release:* labels; release notes generated
   from GitHub + shipped config at cut time (no changelog file); a release
   checklist of one-line, runnable checks.

7. .github/ — issue template for a task, PR template (declared scope vs Owns,
   DoD evidence, docs affected), CODEOWNERS, and CI that restores, builds and
   tests on every push and PR. If the project has randomness, add a determinism
   guard test.

8. .claude/skills/run-task/SKILL.md — install it verbatim from the fenced block in
   docs/build-process.md, and say in the guide that a drifted install is
   reinstalled from that block (skills load at session start).

9. GitHub: create the labels (task, bug, phase:*, lane:*, status:ready /
   in-progress / in-review / approved / rework / blocked / merged / escalated,
   triage:needed, review-round:*, model:*, effort:*, release:*, needs-human,
   local-only, single-instance) and one milestone per phase. Create the first
   four task issues as pointers.

RULES TO ENCODE (each earned its place in the reference project)
  - Status lives only in GitHub labels. A document never carries a status
    snapshot.
  - Contracts live in the repo (the process, the catalogue, the design, the
    investigations) because reviews diff them against a commit; reference
    material that goes stale fast lives in LIVING REFERENCE.
  - Agents never work in the main checkout. One task in flight. The dispatcher
    creates the worktrees; every agent states where it worked (toplevel, HEAD,
    branch/detached, changed files) in its first tool call and final report, and
    the main session verifies that block before relaying a review or merging.
  - Briefs carry the extracted task entry, never a pointer to the catalogue. The
    catalogue is a large file; reading it whole once per agent per round is the
    single biggest avoidable cost.
  - Never read a large file whole. Extract the slice. Scope every search. Keep
    terminal output to counts and file:line. Show diffs, not whole files.
  - One source of truth per idea. If two tools read different instruction files,
    put a one-line scope marker at the top of each saying which tool it governs.
  - A passing test is not a working feature: assert what a person would notice.
    A threshold measured once is a coin toss.

VERIFICATION
Define CHECK, UNIT and FULL. State the expected pass count for each in the
operating guide, read counts rather than the absence of an error, and run FULL
N times before pushing anything that touches primary logic.

KNOWN FAILURE MODES TO DESIGN AGAINST
  - Stale counts and graphs in prose -> generate them, or add a check that fails
    when they drift.
  - A skill that drifts from its documented source -> keep the fenced text as the
    source of truth and a reinstall line.
  - A reviewer skill that forks into the wrong directory and reviews the wrong
    commit -> reviewers sweep the diff inline; every finding must name a file in
    the PR diff; the dispatcher discards any review that does not.
  - Fixtures that drift from committed expectations -> resolve them by name, keep
    the expected table generated, and make the absence of a prerequisite an
    explicit skip, never a silent pass.
  - Hand-maintained "not yet implemented" lists in shipped output -> they go
    stale one merge at a time; if one exists, a test must fail when a listed
    feature starts working.

ACCEPTANCE
  - The five documents exist and are consistent with each other.
  - gh shows the labels, the milestones, the four task issues, and the skill
    installed from its fenced block.
  - CHECK/UNIT/FULL are green on the empty project, with their expected counts
    written down.
  - docs/task-catalogue.md defines T01 (build scaffolding + CI), T02 (core domain
    model + serialization round-trip), T03 (engine seams: RNG, turn pipeline,
    command dispatch, events), T04 (fixtures/test corpus), with the rest of the
    plan outlined but unscheduled.

WORK ORDER
  1. Read the reference documents listed above.
  2. Ask me the open questions the fill-in block cannot answer.
  3. Write the deliverables on a branch; open a PR.
  4. Get that PR reviewed before anything else is built (the process reviews its
     own introduction), then stop and show me.