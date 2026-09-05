# Agent workflow

Every task moves through the same four beats, each backed by a skill from
this collection (or installed alongside it — see [Skill sources](#skill-sources)).
Drop this file into a repo as `AGENTS.md` and fill in the repo-specific
callouts; it also governs work in this repo itself.

## Workflow

1. **Isolate — `/new-feature`.** Every new feature starts in a fresh Git
   worktree branched from `origin/main` so agents can work in parallel
   without conflicts. Never build on `main`.
2. **Build — `/code-structure` under `/ponytail` (full).** Write code to
   the service-layer architecture: actions/boundaries orchestrate the
   "why/when", a service layer owns the reusable "how", with explicit
   inputs and structured returns. Climb the ponytail ladder first: YAGNI,
   reuse existing code, stdlib/native before new code or dependencies,
   shortest diff that fixes the root cause. Precedence where they pull
   apart: ponytail governs *scope* (don't create unneeded code);
   code-structure governs *placement* once repetition across 2+ callers is
   real (extract to the service layer. Do not duplicate, but do not
   over-abstract for a single caller either). The bar is top-notch,
   greploop-clean code: anything below that gets reworked in the Ship
   beat until Greptile reports 5/5.
   Ponytail's terse-output rule applies to code, not to required
   artifacts. PR bodies, evidence reports, and checklists stay complete
   (Prove and Ship beats win over brevity).
3. **Prove — `/evidence-driven-testing`.** Verify with the repo's checks
   plus runtime evidence. Capture the **before** state while reproducing the
   issue — prior to fixing it, when it is cheapest — and the **after** once
   the change works.
4. **Ship — `/before-and-after`, then `/greploop`.** Open the PR with
   before/after proof embedded in the description (screenshot or video
   whenever the change has a visible surface; measured numbers or output
   pairs when it doesn't). Run `/greploop` — or `/greploop-apps` when the PR
   exceeds Greptile's file-count limit — until Greptile reports **5/5 with
   zero unresolved comments**. Finish by presenting the PR URL.

Ship-beat notes:

- `/before-and-after` drives the `@vercel/before-and-after` CLI. `--markdown`
  uploads the pair and prints a PR-ready table; it also accepts existing
  PNGs, so evidence gathered while developing can be reused as-is.
- In containers/VMs where Chrome fails with "No usable sandbox", set
  `AGENT_BROWSER_ARGS="--no-sandbox"` for the capture command.
- The default upload host (0x0.st) is public — fine for ordinary UI shots;
  pass `--upload-url` for anything sensitive.

## Writing for humans

Run `/unslop` over anything a person will read, before you commit, post, or
send it: commit messages, the PR title and body, README and doc edits, code
comments, and the closing reply. It strips AI tells (em dashes, filler,
hedging, chatbot phrases, puffery, bold-label lists) and replaces fancy
words with plain ones and passive voice with active. Apply it to text you
wrote or changed, not to prose you didn't touch.
In this workflow the split is explicit: `/ponytail` governs code scope,
`/unslop` governs prose. (Ponytail's body names Caveman for terse prose.
Caveman is not installed here, so prose means `/unslop`.)

## Evidence in PRs, never in the repo

Before/after screenshots, screen recordings, and any other visual proof
**belong in the PR description** (uploaded and embedded as image URLs /
markdown), **never as files in the repo**. The repo keeps code, not media.

- **Do not** commit `.png`, `.jpg`, `.jpeg`, `.gif`, `.webm`, `.mp4`,
  `.mov`, `.webp` files as part of a feature/fix PR — even when they
  illustrate the change. Upload them via the `before-and-after` markdown
  flow or another host, then attach the resulting links to the PR body.
- Keep raw captures under a gitignored folder (`.artifacts/`, `evidence/`,
  `screenshots/`, etc.) so they stay local for reuse but never reach git.
- When a screenshot absolutely must live next to source (e.g. a docs page
  that renders an image at build time, a fixture that the test suite reads),
  justify it in the PR description and confirm it is tracked in the repo's
  long-lived asset policy — this is the exception, not the rule.
- The same applies to evidence-driven recordings (`evidence.mp4`,
  `report.md`, `manifest.json`): they are PR attachments, not commits.

## Multi-agent rules

- Never commit directly to `main`.
- One worktree and one branch per task and per agent — never reuse or modify
  another agent's worktree, branch, or uncommitted work.
- **Scope check** before starting: skim open PRs' changed files
  (`gh pr list`, `gh pr diff <n> --name-only`) and look for uncommitted work
  in shared checkouts. On overlap, stop and ask for direction.
- Never force-push to `main` — and never plain `--force` anywhere; only
  `--force-with-lease`, only on your own task branch.
- Resolve lockfile conflicts by regenerating, never by hand-merging.
- Worktrees don't isolate shared resources: confirm a dev-server port
  answers *your* process before trusting it, and don't run schema
  experiments against a shared database.
- If a conflict can't be resolved confidently, stop and report instead of
  guessing.

## Completing a task

1. Keep changes limited to the assigned task.
2. Self-review against the top-notch bar: no duplication across flows,
   explicit params, structured returns, no god/leaky services, and nothing
   the ponytail ladder would cut (reuse/stdlib-native first, shortest
   root-cause diff). Then run the repo's checks
   *(repo-specific: list the exact commands here)*.
3. Assemble the evidence captured along the way into before/after pairs.
4. Commit with a clear message, rebase onto the latest `origin/main`, and
   rerun the checks.
5. Push (`git push -u origin <branch>`; after rebasing an already-pushed
   branch, `--force-with-lease`).
6. Open the PR. The body must explain what changed, how it was tested (every
   claim backed by evidence), before/after proof, and any risks or follow-up
   work. Run the title and body through `/unslop` before posting.
7. Run `/greploop` (or `/greploop-apps`) until **5/5 with zero unresolved
   comments**.
8. End by presenting the PR URL.

Do not merge the PR unless explicitly instructed. Keep the worktree until
the PR is merged or closed.

## Repo-specific sections to add

When dropping this file into a project, append what agents need to execute
the beats there: commands & checks, hard invariants (security and
architecture rules), an environment quick reference, local test
infrastructure (stubs, fixtures), and anything that can't be tested locally.

## Skill sources

| Skill | Source |
|---|---|
| `new-feature`, `code-structure`, `evidence-driven-testing` | this repo |
| `before-and-after` | this repo, vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) (or `npx skills add vercel-labs/before-and-after`) |
| `greploop` | this repo, vendored from [greptileai/skills](https://github.com/greptileai/skills) |
| `greploop-apps` | this repo (local variant of greploop for huge PRs; no separate upstream) |
| `ponytail` | this repo, vendored from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) (MIT, license included in the folder) |
| `unslop` | this repo, vendored from [cursor/plugins (pstack)](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop); frontmatter edited so agents apply it unprompted (`disable-model-invocation` dropped, description scoped to text the agent writes or edits for people), body untouched |
