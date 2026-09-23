# Prompt: bootstrap a repository with this harness and planning scaffold

You are setting up a new repository's **process scaffold**, not its product.
Do not write feature code this session. Deliver these files, then open a PR
for them and have a fresh-context reviewer review it — the scaffold goes
through the process it defines.

Repository layout to create:

    PLAN.md                      # the plan + the shared process (§7)
    AGENTS.md                    # OpenCode harness (implementer + reviewer)
    CLAUDE.md                    # Claude Code harness (implementer + reviewer)
    .github/workflows/check.yml  # one CI job per gate
    tools/                       # test runner and check script skeletons

## 1. PLAN.md — one plan document, and the process that builds it

The project has exactly one plan; a second copy goes stale. Two halves.

**The plan proper**
- §0 "Decisions that are the owner's to make": a table of decision | default |
  what it changes. Every decision has a default so work can proceed without
  waiting for the owner.
- Design sections: scope, architecture, files, interfaces, iteration list.
  Each iteration is independently shippable, has a one-line "Done when", and
  ends with its gates green.
- Measurements and lessons go back into this document, each followed by how it
  was obtained — otherwise it is a claim, not a measurement.

**§7 "How this gets built" — the shared process**
- §7.1 One iteration per session, one implementer per harness. No orchestrator;
  the owner starts each iteration. Each harness file points here for process
  and adds only tool-specific process; neither harness applies the other's.
- §7.2 Model and effort per iteration: name the high-effort iterations (the
  ones that can fail quietly); don't drop tiers on the cheap ones.
- §7.3 The reviewer: every PR gets one review from a **fresh context** (a new
  session/subagent that has not seen the work), high effort, same tier as the
  builder. The reviewer receives the plan, the diff, and the check output
  pasted in the PR; it checks, in order: rules against the spec; implementation
  against the agreed design; that the check ran on this commit and every
  assertion still names a defect; the "Done when" item by item; language
  conventions. It reports; it does not fix. The builder fixes in the same PR
  and the reviewer looks once more; a disagreement goes to the owner, never
  into a silent merge.
- §7.4 GitHub at the lowest useful ceremony:
  - One PR per change; the unit of work, review and CI. The body has four
    parts: what was built; the "Done when" as a ticked list; the check output,
    verbatim; what was left out and why.
  - **A PR that completes an issue says so.** After the four parts, the body
    ends with a footer of one `Closes #N` line per issue the PR completes —
    design issue or defect issue. The footer is metadata, not a fifth part,
    and it stays outside the verbatim check-output block. A reference is a
    link; only the keyword closes. A PR that does not complete an issue (a
    first pass, a partial fix) adds no line for it.
  - CI on every PR and every push to the default branch; a red check does not
    merge; nothing is skipped or quarantined to get to green.
  - A DESIGN issue before every implementation change: problem, findings with
    `file:line`, design, open questions. The reviewer posts an explicit
    **AGREE** on it; do not implement before that. Otherwise issues are
    defects found by using the thing, labelled `defect`, each closed by the PR
    that fixes it *and* adds the check that would have caught it (written
    against the broken state first).
  - No project board, no milestones; the plan is PLAN.md.
  - A PR does not merge while its review is still running — if the owner asks
    and a review is out, say so and what the last one found, then let them
    decide with that in hand.
  - Commit messages: one line, English, imperative, no ticket numbers.
- §7.5 What outlives a session: a decision goes to §0, a rule to §7.7, what a
  stranger needs to SPEC.md. Sessions end with a handoff: **Completed** (what
  is now true, with verification), **Files / decisions** (paths touched,
  decisions and why), **Next**. An external reference you forked from is a
  snapshot with a date; record the commit in the plan.
- §7.6 The owner's part: starts iterations, answers §0 defaults, uses the
  product and files what they find as `defect` issues, merges.
- §7.7 Working rules, which every harness follows and reads before changing
  anything:
  - Paths to read, paths to ignore, secrets never to touch.
  - **Change the smallest thing.** Targeted reads and diffs over whole files;
    focused edits; show diffs, not files.
  - Keep command output short; prefer the project's own scripts; cap and tail.
  - **Gates**: a table of each gate with its local command and its CI
    counterpart. Run the full suite (more than once for primary-logic changes)
    before pushing; a red gate does not merge.
  - "After any change to X, run gate Y" rules for each surface, non-optional.
  - The project's silent-failure rules — the constraints that do not error
    when broken — stated as derived, never hard-coded.
  - Conventions: language of user-facing text vs comments/commits/docs;
    build-step and dependency policy.
  - Principles: reproduce every finding — and your own claims — before acting;
    for each passing check say what it would have caught had the code been
    wrong; a passing test is not a working feature; a threshold from one
    measurement is a coin toss; flag out-of-scope defects rather than fixing
    them silently.

**On the check suite** (whatever plays the role of tests): every assertion
names the defect it was written for; a new assertion is run against a
deliberately broken version of the thing it covers — **in the direction the
change can actually go wrong** — before it is trusted. The rule that keeps
being relearned: *an assertion only sees the states the check renders* — when
the product gains a state, the check gains the row that puts it there.

## 2. AGENTS.md — the OpenCode harness

Thin; points at PLAN.md §7/§7.7 and adds only this harness's process:
- **Read first**: PLAN.md §7.7 and §7 before changing anything.
- **Roles**: Implementer = `<implementer model>`; writes the design, the code
  and the tests; replies on GitHub signed `— Implementer (<model name>)`.
  Reviewer = `<reviewer model, high variant>`, invoked as a subagent in a
  fresh context and given an explicit model id; verifies against the real code
  rather than the description; posts its verdict on GitHub signed
  `— <reviewer name> (<model name>, high)`. The reviewer posts through the
  owner's account, so the signature line is the only authorship marker. A
  BLOCK is not overridden by the implementer — it goes to the owner.
- **Two stages**: (1) DESIGN issue → reviewer comments → iterate to an
  explicit AGREE → only then implement; (2) implementation on a branch → PR
  referencing the issue → reviewer reviews the PR against the agreed design →
  iterate to an explicit AGREE → the owner merges. Every change, no size
  floor.
- **Bootstrap**: a change that introduces or edits AGENTS.md or CLAUDE.md
  follows the same two-stage process.
- **Verification**: the implementer runs the gates; the reviewer reproduces
  what it can.

## 3. CLAUDE.md — the Claude Code harness

Thin, same "read PLAN.md §7.7/§7 first" pointer, but no cross-harness roles:
Claude implements, a fresh-context Claude session reviews, the owner merges.
State explicitly that neither harness applies the other's process.

## 4. Gates and CI

- Create the CI workflow now: every PR and push to the default branch, one job
  per gate (a test job, a check job).
- Make the commands exist from day one (test runner skeleton, check script
  skeleton) even if they assert little.
- If your tool supports skills or slash commands, add one for operating the
  check suite: what it covers, how to add an assertion, how to read a failure.

## 5. The kickoff prompt for later sessions

Write the session template into PLAN.md §7.1, for example:

> Do iteration N of `PLAN.md` in `<owner>/<repo>`. Read `PLAN.md` in full
> first, then `<reference material: sibling repo, design docs, previous PR>`,
> then the previous iteration's pull request. Work on a branch named
> `iteration-N-<slug>` off the default branch. Stop at the iteration's "Done
> when": do not start the next one. Finish with every check green, commit,
> push, and open a pull request with the description in §7.4.

## What not to do

- Do not write SPEC.md yet; it is the handover written when the thing ships,
  from PLAN.md and what actually got built.
- Do not invent process you do not need: no project board, no milestones, no
  status files, no CI beyond the gates.
- Do not leave the process only in chat. If a session ends with something only
  it knows, that is a defect in the handoff.

## Finish

Open a PR for the scaffold that references this setup issue and ends with the
closing footer. A fresh-context reviewer reviews it to AGREE before the owner
merges; if no reviewer is available, leave the PR for the owner rather than
merging it yourself.

Now ask me for the project brief — name, what it is, primary language and
tooling, the gate