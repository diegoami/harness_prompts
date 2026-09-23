You are bootstrapping a new repository. Before any product code, set up the
harness, planning, review and implementation scaffold below. Ask me for
anything in <angle brackets> rather than guessing. Keep every file short and
owned: one fact has exactly one home.

# 1. The context file (read first by every session)
Create CLAUDE.md as a *context budget*, not a story:
- §0 Canonical sources: which file owns which fact (status, decisions,
  changelog, release process, the current plan) and a table of paths never to
  read/glob/grep (generated data, build output, lockfiles, binaries, vendored
  deps). State the budget in one line: "read this file and the file the task
  names; never read the repo to get oriented."
- §1 What the project is, and an annotated directory map (one line per path).
- §2 Commands: dev, build, test, gates, release.
- §3 The working rules (§3 below).
- §4 How a session hands over: "start by reading docs/HANDOVER.md, then the
  plan it points at."

# 2. The document set (one owner per fact)
- ROADMAP.md — status log, newest first: one entry per release, what shipped;
  then a backlog of raised-but-unscheduled items, each with the date raised and
  the decision that settled it (if any).
- DECISIONS.md — the "why" log, newest first: one heading per decision, the
  options, the choice, the reasoning. Not implementation detail.
- CHANGELOG.md — release-facing notes, one entry per tagged version: a bold lead
  sentence, user-facing bullets first, then "under the hood", then "not covered".
- docs/HANDOVER.md — the snapshot a fresh session reads first: a "where things
  stand" table (branch, deploy, latest release, next, after that), "how to
  resume" with the exact first message, "things only the owner has", "gotchas
  learned the hard way", and coordination notes if several agents share the repo.
- docs/RELEASES.md — the release process: what counts as a batch, versioning,
  the checklist (bump, gates, smoke test, owner OK, tag, package, publish),
  pre-releases, and what a notes entry must say.
- docs/REVIEW_LOOP.md — the review procedure (§4).
- docs/PLAN_<version>.md — one per release (§2b).

# 2b. Planning, per release
One plan file per release, containing:
- a one-paragraph "why this release", and whether product decisions are needed;
- task entries: `### <ID> — <title> · <size> · deps: <IDs>`, then **Why**,
  **Do**, **DoD** (testable), and any **open question** marked 🧑 for the owner;
- an **Order** section: what goes first and why;
- a **Progress ledger** table (task, state, merge commit, notes);
- an **Out of scope** section;
- a **product decisions** table when the owner must choose, with options and a
  recommendation.
Deferred tasks keep their specs in the plan, marked deferred with the reason.
A spike is a task whose DoD is a decision, not a shipped feature.

# 3. Working rules
- One branch per task: branch, code, run the gates, push, open a PR.
- One gates command (`npm run gates`) runs every quality gate in order
  (typecheck, unit tests, lint/format, production build), fails fast, and has a
  `--quiet` mode. Install a pre-push git hook that runs it.
- Every task PR is reviewed by a *different model* (§4) before merge.
- The owner OKs every merge and every tag. Publishing a release the owner already
  asked for needs no second OK, but say plainly in the notes what was not verified.
- Docs are part of the work: a change updates the plan ledger, ROADMAP.md and
  DECISIONS.md in the same PR (or its immediate follow-up), never "later".
- Never hand-edit generated files: edit the source and rebuild. Say which path is
  source and which is output, and make the build reproducible.
- Verify in the real target (browser, device, installer), not only in tests.
- Name the implementing model in the commit trailer, e.g.
  `Co-Authored-By: <Implementer Model> <noreply@example.com>`.

# 4. The review loop (docs/REVIEW_LOOP.md)
- Two roles, deliberately different models: an **implementer** (writes code,
  opens the PR, answers findings) and a **reviewer** (a second model, reviews the
  diff on GitHub). Each signs every comment with its model name.
- Per task: implementer pushes and opens the PR (body says what changed, how it
  was verified, any deviation); reviewer reads the diff and posts an honest review
  ranking findings **blocking / worth doing / nit**, naming file and line for
  anything blocking; implementer answers every finding (fixed, or why not);
  repeat until the reviewer states, in a comment, that findings are resolved or
  explicitly deferred. Silence is not agreement.
- Run the reviewer as a spawned subagent with a *specific* model, told to post
  with the GitHub CLI and to sign its comment; if it cannot post, the implementer
  relays its words verbatim and says who wrote them.
- A declined finding is settled only when the reviewer accepts the reason; if the
  two cannot agree, both record it and the owner decides.
- The loop never replaces the owner's merge OK.

# 5. Handover
Maintain docs/HANDOVER.md so a cold session can pick up: what is done, what is
next with the exact first message ("Read docs/HANDOVER.md, then docs/PLAN_X.md.
Start <ID>."), what only the owner has, and the gotchas. The plan ledger is the
durable task board; the handover is the snapshot.

# 6. Automation (thin, no server)
- `npm run gates` (§3) with a `--quiet` mode, plus the pre-push hook.
- A small CLI (`scripts/task.mjs`) for what you do often: a `doctor` that reports
  stray branches/worktrees/servers, per-task state and log, and the gate runner.
- A version-sync script: one source of truth for the version, propagated to every
  manifest, with a `--check` mode the release gate runs.
- Release scripts: package the installers into a gitignored directory and verify
  checksums; a separate publish step that is dry-run by default and needs an
  explicit confirm flag.

# 7. Bootstrap order
1. Create the repo, the context file, and the document set, with one honest entry
   each ("nothing shipped yet").
2. Add the gates command and the pre-push hook; make them pass before anything else.
3. Write the review-loop file and the release-process file.
4. Write the first plan (even if it is one task) with a ledger.
5. Only then start product code, on a first task branch, through the loop.
6. Whenever a task needs a product decision, put it in the plan's decision table
   and ask; record the answer in DECISIONS.md.