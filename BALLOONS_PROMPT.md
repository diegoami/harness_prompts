You are bootstrapping a new repository with the same working scaffold as
<reference project>. Set it up deliberately, and record each decision in the
files below rather than in this conversation. If you are making this change to
an existing repo, it follows the review process it is creating (section 8).

## 1. Repository shape

- <source dir> — the product. This is the ONLY place the product is edited.
- Any generated or mirrored copy (platform shell, build output, assets copied
  at build time) must be gitignored, and the memory file must say: "never edit,
  never cite; go and read the original".
- <tests dir>, <tools dir>, <docs dir>.
- No build step if the stack allows it; if there is one, it is one command.
- State plainly which paths are the source of truth and which are shadows.

## 2. Memory files — one source of truth per idea

- README.md: what it is; the module map; how to run; how to test; how to
  deploy. Keep the module map complete — an incomplete one drifts.
- AGENTS.md (read by OpenCode): the review process (section 4) and the product
  guidance. Start it with a scope marker:
  "> Guidance for <tool A>. <tool B> uses <other file>."
- CLAUDE.md (read by Claude Code): product guidance and the tool-agnostic
  principles only — NO reviewer-spawning mechanics, because that tool cannot
  spawn a different-family model. Its own scope marker.
- docs/open-work.md: small unfinished jobs, plus any planned feature. Header
  says "Current as of <version>".
- Rules to write down: durable traps and decisions go here, not in private
  notes; list the big files and say "grep the test name, read a window, do not
  read the suite whole"; list secrets and gitignored files as never-read;
  prefer rg over grep -r (gitignore does not stop find/grep).

## 3. Harness — layers, each one command

1. `check`: syntax-check EVERY module, and cross-check that the entry point
   loads exactly the modules on disk. Write it as a script, not a shell list:
   a list drifts, and `node --check glob` only checks the first file and exits
   0 for the rest.
2. `test:unit` (or :scores/:core): the backend/logic against an in-memory mock
   of every external service, so it runs with no network.
3. `test:integration`: drive the REAL app headless (Playwright/Chromium).
   Cover rendering, input, scoring/state, the full round trip, device-pixel
   ratios, resize and rotation.
4. `measure` (a tool, not a test): an operator/bot with human limits — reaction
   delay, aim/error tolerance, a realistic action rate. It asserts nothing; it
   reports outcomes and distributions, and exists to answer questions that
   reading the code cannot.
5. `test`: check, then both suites. This is the gate before a push.

If the product draws or renders, add a test that measures the REAL artefact
(pixels, contrast, timing) rather than a copy of what it claims to draw.

## 4. The review process — two stages, over GitHub

ROLES
- Implementer: <implementer model>. Writes design, code and tests; replies on
  GitHub signed "— Implementer (<model>)".
- Reviewer: <reviewer model, a DIFFERENT family>, high reasoning effort,
  invoked as a fresh subagent with an explicit model id. It verifies against
  the real code, not the description, and posts verdicts signed
  "— <Reviewer> (<model>, high)".
- The reviewer posts through the owner's account; the signature is the only
  marker of authorship.
- A BLOCK is not overridden by the implementer — it goes back to the owner.

STAGE 1 — DESIGN AS AN ISSUE. Before any implementation, open a GitHub issue:
the problem; findings grounded in the code with file:line references, each
claim verifiable; the proposed design (alternatives where there is a real
choice); and explicit open questions, separating OWNER DECISIONS (product calls
the implementer must not make alone) from reviewer requirements. Have the
reviewer review and comment; iterate until the reviewer posts an explicit
AGREE. Do not implement before that.

STAGE 2 — IMPLEMENTATION AS A PR. Implement the agreed design on a branch;
open a PR that references the issue; have the reviewer review it against the
agreed design; fix and iterate until an explicit AGREE. The owner merges.

In a tool that cannot spawn another family (e.g. a single-model CLI), bridge
out for the review: shell out to another model's CLI (e.g. `codex review`, or
`opencode run -m <provider>/<model>`), or fall back to a same-family reviewer
and CALL IT a second pass, not an independent review. Do not pretend.

## 5. Principles (put these in both memory files, tool-agnostic)

- Keep reviewer requirements separate from OWNER DECISIONS; put owner
  decisions to the human with a recommended default.
- Reproduce every finding before acting on it, and your own claims before
  publishing them. When a check fails, suspect your harness first — run old
  and new side by side, because a broken harness shows both columns agreeing.
- For each passing check, say what it would have caught had the code been
  wrong. Never let implementer and reviewer share a blind spot.
- A passing test is not a working feature: assert what a person would notice
  (pixels, contrast, timing), then go and use it.
- A threshold taken from one measurement is a coin toss: take a distribution,
  set the bound outside it.
- Flag out-of-scope defects rather than fixing them silently; if a fix turns
  out bigger than flagged, fix it fully.
- Show diffs, not whole files; do not re-echo a file you just edited.

## 6. Verification gates

- Climb the ladder: check, then unit, then integration.
- Run the full suite N times (start with 8) before pushing anything that
  touches primary logic, and read the pass COUNT, not the absence of a FAIL.
  Re-run after the last edit, never before.
- A change that touches no shipped or timed file cannot move timing; one pass
  plus the diff is proportionate there. Say which level of verification a
  change got rather than implying the higher bar.

## 7. Release (if there is a shippable artefact)

- Bump the version in its own PR: strictly increasing build/version code,
  human version name; the PR body says what it carries.
- Build and sign; verify the built artefact actually contains the merged
  source (grep the packed/bundled copy for a symbol you just added).
- Publish as a GitHub release with a named asset (rename if the tool ignores
  the rename), in the same notes style as the previous release.
- Publishing is outward-facing: never do it without the owner's go-ahead.

## 8. Bootstrap this scaffold itself

The commit that creates AGENTS.md/CLAUDE.md and the harness follows the same
process: open it as a PR and have the reviewer review it to an explicit AGREE.
The process reviews its own amendment.

---
Fill in: project name and directories; implementer and reviewer model ids;
the review tool/bridge; the exact test commands and the run count; whether
there is a build step and a release artefact; and the never-read list.