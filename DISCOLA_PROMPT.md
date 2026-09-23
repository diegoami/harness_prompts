Set up a new repository, <PROJECT>, with the working process below — the scaffold
first, not the product. Before creating anything, list the owner decisions you
need, each with a recommended default, and wait for an answer.

## 1. Two guidance files, one idea each

- `CLAUDE.md` — tool-agnostic: the process in outline, the principles, the
  verification gates, the project rules. It must not name a reviewer mechanism.
- `AGENTS.md` — the OpenCode-specific review process: roles, the two stages, the
  bootstrap rule. It points at `CLAUDE.md` for principles.
- One source of truth per idea: process in `AGENTS.md`, principles and gates in
  `CLAUDE.md`. The change that introduces either file is reviewed like any other
  change — the process reviews its own amendment.
- Keep the model ids in `AGENTS.md` current: verify them against the provider's
  model list instead of trusting the file.

## 2. Roles and the loop

- **Implementer**: `<implementer model id>`. Writes the design, the code and the
  tests; signs GitHub comments "— Implementer (<name>)".
- **Reviewer**: a different model, `<reviewer model id>#high`, invoked as a
  subagent in a fresh context and given its model explicitly. It verifies against
  the real code, never against the description, and signs its verdict
  "— Reviewer (<name>, high)".
- The reviewer posts through the owner's GitHub account, so the signature line is
  the only marker of authorship.

Two stages, both on GitHub:

1. **Design.** Write the proposal as an issue: problem; findings with `file:line`
   references; the design; open questions. The reviewer posts an explicit
   **AGREE** or **BLOCK**. Do not implement before AGREE. Answer a BLOCK with
   numbered corrections and ask for re-review; iterate to AGREE.
2. **Implementation.** Implement on a branch and open a PR that references the
   issue. The reviewer verifies the PR against the agreed design, not against the
   PR description. Fix and iterate to AGREE. **The owner merges.**

Rules that make it work:

- A BLOCK is never overridden by the implementer — a disagreement goes to the
  owner.
- Keep reviewer requirements separate from **owner decisions**, and put owner
  decisions to the human with a recommended default.
- Reproduce every finding before acting, and your own claims before publishing
  them; when a check fails, suspect your harness first.
- For every passing check, say what it would have caught had the code been wrong,
  so implementer and reviewer never share a blind spot.
- A passing test is not a working feature: assert what a person would notice.
- A threshold from one measurement is a coin toss.
- Flag out-of-scope defects rather than fixing them silently.
- Show diffs, not whole files.

## 3. Verification gates, wired into the repo

- One command per class of defect the project can actually ship, exposed as npm
  scripts: `check` (the expensive end-to-end check), `test` (fast deterministic
  unit tests), `verify` (check then tests). CI runs them on every pull request
  and every push to the default branch.
- Every threshold in a check is calibrated against a defect that actually
  shipped, and the file says which.
- Run a gate after any change to the surface it guards. Run the full suite three
  times before pushing anything that touches primary logic, and read the pass
  **count**, not the absence of a failure.
- One skill per gate in `.claude/skills/<gate>/SKILL.md`: what it covers, how to
  run it, how to read a failure, when it is required. The description is the
  trigger — write it so it fires on the right changes.

## 4. Context and token discipline

- "Read this much, and no more": an explicit list of what to inspect normally and
  what to ignore; never read or paste secrets.
- Smallest change: search first, read only the range you need, edit with focused
  replacements, show `git diff` rather than reprinting files.
- Cap command output; prefer the repository's own commands over ad-hoc
  exploration; use `node --check` for a syntax check instead of running a script.
- Fresh session after each completed logical unit — a merged PR, a finished fix —
  carrying a short handoff: **Completed** / **Files and decisions** / **Next**.
- Durable facts belong in the repository (the docs, the guidance files, the PR
  body), not in the conversation.

## 5. Docs as the project's memory

- `<SPEC>.md` — what exists, and the properties that must not be given up.
- `<ROADMAP>.md` — what a larger change would take, as independently shippable
  iterations, with the original reasoning kept even after the decision is made.
- One companion doc per delivery target or integration, recording the decision
  and its date, spike results, what is deferred and why, and the next steps.
- Cite repository facts as `file:line`; mark anything not sourced from the repo.

## 6. Issue and PR mechanics

- Issues: title `[Proposal] …`; problem, findings, design, owner decisions,
  acceptance criteria as checkboxes, out-of-scope named explicitly.
- PRs: reference the issue; the body states what changed and what was verified —
  the commands run and what they proved.
- Outward-facing tooling (publishes, uploads, migrations) is dry-run by default
  and needs an explicit `--confirm`; it verifies its own inputs and fails loudly
  rather than shipping a half-built artifact.

## Deliverable

Create the scaffold files above for <PROJECT>, with placeholders where the
project's own gates, docs and model ids are not yet known. Then stop and ask the
owner decisions, each with a recommended default. Do not write product code in
this change.