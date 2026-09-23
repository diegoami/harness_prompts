## Your task

I want a new project to have the same working scaffold as `diegoami/Scopetta`: a two-stage, cross-model review harness, a split agent-guidance set, and a verification-gate discipline. You will **build that scaffold in this repo, and then demonstrate it once** on a real (small) change, so the process is proven rather than described.

The reference implementation is `https://github.com/diegoami/Scopetta` — read `AGENTS.md`, `PRINCIPLES.md`, `CLAUDE.md`, and `PLAN.md` §7 there. **Do not copy them wholesale**: they carry Scopetta's own facts (its card rules, its gates, its reviewer id). Mine the *structure* and the *rules*, and fill in the blanks for this project. Where I must decide something, ask me — see "Owner decisions" below.

## Step 0 — interview me, briefly

Before writing anything, ask me and record the answers:

1. **The gates.** What commands actually verify this project? (A fast deterministic test suite, and whatever expensive end-to-end check applies — a UI check, an integration suite, a build.) I need the *real* ones; if there are none yet, say so and propose them as the first task.
2. **Project conventions.** What is frozen and what is mutable? What does the agent most often get wrong?
3. **Reviewer model.** Which model reviews, and does it differ in family from the implementer?
4. **Shared hosting.** Is this on GitHub under my account, with `gh` authenticated?

## Step 1 — the three guidance files, with one home per idea

Create these, each starting with a **literal first-line blockquote marker** naming its audience and its siblings:

- **`PRINCIPLES.md`** — what **both** tools read. Owns: the working principles and habits; and a **single source of truth** section containing an ownership map (which file owns which idea), the link-don't-restate rule, and the **non-trivial test** (below).
- **`AGENTS.md`** — the OpenCode/host-agent guidance. Owns: the **review process** and the **verification gates**. Scope marker: *"Guidance for OpenCode. Claude Code uses CLAUDE.md. Shared principles: PRINCIPLES.md."*
- **`CLAUDE.md`** — the Claude Code guidance. Owns: the **project's own rules** only. It **must not contain the reviewer-spawning mechanism** — Claude Code cannot spawn another family, so its process is lighter and it says so. Scope marker naming `AGENTS.md` for the process and `PRINCIPLES.md` for the principles.

**The one-home rule is the spine of the whole thing:** an idea lives in exactly one file; every other file **links** to it and never restates it. A contradiction between the files is a `defect` to fix in the change that found it. This rule will catch its own violations later — it did, four times, in Scopetta.

## Step 2 — the review process, in `AGENTS.md`

- **Two stages.** A non-trivial change starts as a **GitHub issue holding a proposal**; the reviewer's verdict is **posted and signed on that issue** and the stage ends on an explicit **AGREE**. Then the **pull request** is reviewed the same way to an explicit **AGREE** there. The PR **links the design issue, states which revision the AGREE was given on**, and carries **`Closes #n` on its own line** so the record closes itself.
- **A different model family, fresh context, high effort, explicit model id**, invoked as a subagent. Keep a **table of implementer and reviewer with the rule that the family difference is the invariant and the ids are merely the current assignment** — whoever changes an assignment updates the table in the same change.
- **The verdict is a signed comment, not an approval.** Signature convention: `— <reviewer display name> (<model id with variant>), reviewer`. Record plainly that it is attribution under my GitHub account, **not cryptographic provenance**, and that `--approve` is impossible because it is my own PR.
- **`AGREE`, materiality, re-review.** Any change after an AGREE invalidates it and requires a fresh signed verdict — *except* commit-message/whitespace/typo edits that change no behaviour, no assertion and no process text. The **initial** verdict for each stage is a **new session**; a re-review after fixes may continue that session.
- **A BLOCK stands, and is scoped.** Write these two as one rule, because they are one idea: a required change must be **necessary to the proposal's aim, its correctness, or its verification** — and **the reviewer decides whether a finding is necessary; the verdict rests on that; neither party's label changes it**. An in-scope finding may be a BLOCK, and the implementer satisfies it or takes it to me, never merges around it. A finding **outside** that scope does not block: the reviewer **posts it marked out of scope**, the implementer **lists it for me in the chat with a proposed issue for each**, and **I decide which become issues** — recorded on the issue/PR, and **my silence does not block**. Growing to take a necessary finding is a **withdraw-and-re-scope**, recorded, invalidating the AGREE. **An owner decision sets a value or a boundary and never reclassifies a finding.**

## Step 3 — the trivial route, and the gates

- **The non-trivial test**, in `PRINCIPLES.md`: a change is non-trivial if it can change **(a)** observable behaviour, **(b)** what any check measures or asserts, **(c)** the design or process a builder follows (including these guidance files and the plan/spec documents where they state design), or **(d)** user-facing copy. Plus a **conservative floor** — a path list the builder checks instead of tracing imports — and a **typo exception** so a pure typo stays trivial *even in the guidance files*.
- **A trivial change takes neither stage**: no issue, no PR, no verdict; it may be committed straight to the default branch, and it still runs every gate its diff can affect.
- **The gates**, in `AGENTS.md`, with the **unit of a run** stated: the unit is a **push's tip**; the cheap suite runs on every push; the expensive check runs on a push whose **measured-input tree** changed (define it: the transitive runtime inputs of the expensive command) or after a rebase, and a rebase **always** re-runs it. Separate **local** from **CI** and say which is the merge gate.
- **Assertion removal and threshold retuning**: adding, changing or removing an assertion runs the mutation/break harness, and the break must trip **the assertion written for that defect**; a removed assertion needs a replacement or a recorded reason plus a mutation showing which remaining assertion covers it; a retuned threshold is re-verified against the commit that introduced the bug, with a **second** measurement named.
- **A change to the guidance files is itself non-trivial by definition** — the process reviews its own amendment.

## Step 4 — it must actually run, once

Do not hand me a scaffold I have to trust. **Demonstrate it end to end on one small real change**, chosen with me:

1. Open a **design issue** with the proposal and any **owner decisions** recorded with a recommended default, the reason, and an owner-decision mark.
2. Spawn the reviewer as a subagent with the explicit model id and let it **post a signed verdict on the issue**. Iterate to a signed **AGREE** — expect BLOCKs; they are the point.
3. Implement on a branch, open the PR (with `Closes #n` on its own line), and get a **fresh** reviewer session to **post a signed verdict on the PR**.
4. Merge on green, confirm the default branch is green, and report.

If the demonstration exposes a contradiction in what you wrote, **fix the guidance in that same change and say so** — that is the process working, not failing.

## Owner decisions (ask; do not choose for me)

Present each with a recommended default and your reason, and **record my answer on the issue or PR**, because conversation is not a record: the reviewer model and family; the gate commands; whether the expensive check is required locally or CI-only; the default branch and whether trivial changes go straight to it; anything else that is a name, a licence, a scope, or a default.

## What "done" looks like

`PRINCIPLES.md`, `AGENTS.md` and `CLAUDE.md` exist with literal markers and **no duplicated rule**; the two-stage process and the scoped BLOCK rule are written down; the gates and their run unit are stated; the scaffold has been **used once** to land a real change to a signed AGREE on both stages; and the default branch is green with the tree clean. Report what you built, what the demonstration cost, and anything you had to leave undecided.

## Notes on using it

**Adapt the gate vocabulary.** Scopetta's split — a fast `node --test` suite and a ~25-minute `check_ui.mjs` — is one shape. If your project has a type check, a build and an integration suite, that's your cheap tier / expensive tier; the *rules* transfer, the commands don't.

**The cheapest way to start is Step 4.** If you'd rather not front-load the whole scaffold, run the scaffold's core — the two stages, the signed verdict on GitHub, the scoped BLOCK — on one small change first, and let the three files **accrete** from what the review actually demanded. That's how Scopetta got there: the harness was amended **three times in one day**, each time because the process, run in anger, contradicted itself. The documents are downstream of the practice.

**Two things I'd flag from that experience.** First, the failure mode is always the same: the same rule written in two places, drifting. The ownership map plus "link, don't restate" is what stops it, and it only works if the map is **authoritative** — a contradiction is a defect, not a judgement call. Second, resist writing the trivia rule into the guidance files: `Closes #38` is open in Scopetta precisely because I wrote "the PR carries `Closes #n`" without checking that GitHub requires it on its own line. State the mechanism you have **verified**, not the one you believe.