# agent-checkpoints

Reusable review checkpoints for AI coding agents (Claude Code, Codex, Cursor, Gemini, Aider). Each checkpoint produces a structured review with confidence ratings and P0/P1/P2 findings.

### Starting an idea

Before code or specs exist, figure out if it's worth building.

- [`planning/idea-validate.md`](planning/idea-validate.md) — **adaptive orchestrator**: classifies the idea (new product / new feature / internal tool / research), applies JTBD + requirements-interview + competitor scan + risk surface + resource fit as relevant, runs a verifier loop, and produces a GO / NO-GO / GO-WITH-MODIFICATIONS verdict with v1 scope or kill criteria. *Most ideas die in the gap between "sounds plausible" and "anyone will actually use it".*
- [`ux/jtbd.md`](ux/jtbd.md) — name the actual job a user is hiring this for. *Most plans skip "is anyone going to want this?"* (The orchestrator runs this when relevant; fire it standalone for a deeper single-lens pass.)
- [`planning/requirements-interview.md`](planning/requirements-interview.md) — clarify ambiguous asks one question at a time. *Vague specs produce vague code.*
- [`marketing/distribution-strategy.md`](marketing/distribution-strategy.md), [`marketing/go-to-market.md`](marketing/go-to-market.md) — channel and GTM thinking. *If no one finds it, it doesn't matter that it works.*

### Planning

Once you have a plan, pressure-test it.

- [`planning/plan-critique-1x.md`](planning/plan-critique-1x.md), [`planning/plan-critique-3x.md`](planning/plan-critique-3x.md), [`planning/plan-critique-5x.md`](planning/plan-critique-5x.md) — adaptive cycle that classifies the plan and pulls in JTBD/UX/validation. *Plans look reasonable until you stress them.*
- [`planning/principal-critique.md`](planning/principal-critique.md) — adversarial senior-engineer pass. *Someone needs to be the skeptic on every plan.*
- [`planning/team-assembly-1r.md`](planning/team-assembly-1r.md), [`planning/team-assembly-2r.md`](planning/team-assembly-2r.md), [`planning/team-assembly-3r.md`](planning/team-assembly-3r.md), [`planning/team-assembly-5r.md`](planning/team-assembly-5r.md) — assemble personas, run multi-round discussion. *Solo plans miss what a domain expert would catch.*
- [`validation-criteria.md`](validation-criteria.md) — define observable "done" before starting. *"Make it work" isn't a test.*

### Reviewing

Once code is written (by you or an agent), verify it.

- [`dev/pr-review.md`](dev/pr-review.md) — **adaptive orchestrator**: classifies the diff and pulls in the security/perf/UX/migration/quality-hunt lenses as needed; dedupes findings into one P0/P1/P2 list. Trivial diffs degrade gracefully to a focused pr-review pass. *"LGTM" misses real bugs; running 5 separate reviews after every PR is what gets skipped when tired.*
- [`qa/security-gate.md`](qa/security-gate.md) — paranoid attack-surface analysis on the diff. *Every input is potentially hostile.* (The orchestrator runs this when relevant; fire it standalone for a deeper single-lens pass.)
- [`qa/performance-profiler.md`](qa/performance-profiler.md) — find what breaks under load. *Big-O matters at 100x.*
- [`qa/quality-hunt.md`](qa/quality-hunt.md) — find similar bugs elsewhere. *The bug you fixed is rarely the only instance.*
- [`qa/manual-qa-checklist.md`](qa/manual-qa-checklist.md), [`qa/quick-smoke-test.md`](qa/quick-smoke-test.md), [`qa/browser-qa.md`](qa/browser-qa.md) — testing at three depths. *"Did you test it?" needs concrete steps.*
- [`ux/ux-critique.md`](ux/ux-critique.md) — UI/UX review. *Confident-looking UIs ship with broken affordances.*
- [`dev/evaluate-feedback.md`](dev/evaluate-feedback.md), [`dev/fix-issues.md`](dev/fix-issues.md) — work through feedback and fix. *Feedback without triage turns into a TODO graveyard.*

### After shipping

Close the loop.

- [`dev/ship-and-cleanup.md`](dev/ship-and-cleanup.md) — **adaptive orchestrator**: pre-flight check, classifies the work, runs merge + worktree cleanup, refreshes drifted docs when relevant, captures lessons for non-trivial work, final state check. Trivial chores skip docs+lessons and you get effectively the original merge+cleanup. *Branch detritus accumulates fast; docs drift silently; lessons evaporate within hours of merging.*
- [`dev/doc-cleanup.md`](dev/doc-cleanup.md) — refresh canonical docs against the changes. *Drifted docs are worse than missing ones.* (The orchestrator runs this when relevant; fire it standalone for a deeper pass.)
- [`learning/lessons-learned.md`](learning/lessons-learned.md) — extract generalizable patterns. *Same mistake twice is a process failure.* (Same — composed by the orchestrator; available standalone.)

Copy any body into your AI chat. Nothing to install. Full file-level list in [INDEX.md](INDEX.md).

## Folder layout

```
agent-checkpoints/
├── planning/                  plan critiques, requirements, team reviews
├── dev/                       PR review, fix issues, doc cleanup, ship
├── ux/                        JTBD, UI/UX critique, friction-to-fix
├── qa/                        security, performance, manual tests, smoke
├── marketing/                 distribution, GTM, campaigns, audits
├── learning/                  lessons learned
└── validation-criteria.md     cross-cutting (used by most plans)
```

## How to use them

Copy the body of any prompt and paste it into your chat. That's the simplest path.

If you have tooling that pulls prompts from URLs, the raw URL is `https://raw.githubusercontent.com/ShivamGupta42/agent-checkpoints/master/<folder>/<name>.md`.

If you want to change them to match how you think, fork the repo and point your tools at your fork.

## Chains and gaps

Most prompts are run manually today — pick one, paste, run. A growing set are orchestrators that classify the work and compose other prompts on demand:

- `idea-validate` (`::IV`) classifies the idea, then pulls JTBD + requirements-interview + competitor scan + risk surface + resource fit as needed; outputs a GO / NO-GO / MODIFY verdict with v1 scope or kill criteria. Use BEFORE you commit to building.
- `plan-critique-3x` classifies the plan, then pulls JTBD + UX + validation as needed
- `pr-review` (`::RO`, "review only") classifies the diff, then pulls security + perf + UX + migration-safety + quality-hunt as needed, dedupes findings, runs a confidence-gated verifier loop (max 3 iterations), and produces one P0/P1/P2 list — one-shot, no fix
- `review-and-fix` (`::RF`) wraps `::RO` in a loop: review → fix P0/P1 → re-review → repeat, until zero P0/P1 remain or the 5-cycle cap is hit. P2s are reported as "deferred — not blocking ship" and are never fixed inside this loop by design
- `ship-and-cleanup` (the upgraded `::W`) pre-flight-checks, classifies the work, runs merge + worktree cleanup + adaptive doc cleanup + lessons-learned + final state check
- `team-assembly-3r` proposes 3–5 personas, runs N rounds of discussion, outputs sequenced tasks

```
       pr-review (orchestrator)
             │
             │  classifies the diff, then pulls:
             ▼
   ┌─────┬─────┬───────────┬───────────────┬──────────────┐
   ▼     ▼     ▼           ▼               ▼              ▼
   sec   perf  ux-critique migration-safety quality-hunt  (always
   gate                                                    just runs
                                                          pr-review
                                                          spec)
            │
            ▼
   dedupe → cross-lens patterns → verifier loop (max 3) →
   proof-of-understanding for P0 → unified P0/P1/P2 output
```

All three chains identified in the original "worth building next" list have shipped (`idea-validate`, `pr-review` orchestrator via `::RO`, `ship-and-cleanup` orchestrator via `::W`). With `::RF` layered on top of `::RO`, the review side now has both a one-shot and a self-closing loop. The workflow loop is closed: idea → plan → review → ship.

Gaps I notice when actually using these on real projects:

- **Observability review** — does the new code emit useful logs, metrics, traces?
- **Architecture decision record** — capture why X over Y in the moment, not after the fact
- **Estimation prompt** — bound the work realistically before committing
- **Rollback plan review** — what's the fast revert when the rollback signal fires?
- **Rollback plan review** — what's the fast revert when the rollback signal fires?

Issues and PRs welcome.

## Design choices

The prompts share five traits:

**Adversarial by default.** They ask the model to push back, rate confidence, and STOP rather than guess when context is thin. Sycophantic output is what I'm trying to avoid.

**Explicit confidence scale.** Whenever a prompt asks for confidence, it's on a fixed scale (1–3 guessing, 4–6 informed but unverified, 7–8 verified by reading code, 9–10 verified with test or external source). Same anchor everywhere.

**P0/P1/P2 triage.** Findings get sorted into one of three buckets. Easy to scan, easy to act on.

**Skip with a reason; never fabricate.** When a section doesn't apply, the prompts say so instead of padding to look thorough. Catches a lot of slop.

**Inputs flow forward.** Every step's inputs are produced by earlier steps, never later ones. If a step references something a later step is supposed to produce, it's a bug — either reorder, or insert the missing producer. Surfaced after auditing 6 ordering bugs that all had the same shape (downstream step depending on input that hadn't been produced yet). Worth using as the diagnostic when designing or critiquing any multi-step prompt.

## Where these come from

These are synced from my espanso config (a text-expansion tool I use locally). I edit the espanso file, run a small script, and the script writes each trigger to a file here.

```
   espanso config (private)              this repo (public)
   ────────────────────────              ──────────────────
   base.yml                              planning/*.md
     ::P1, ::P3, ::J, ::QV, ...   ──►   dev/*.md
                                  sync   ux/*.md
                                         qa/*.md
                                         ...
   (source of truth)                     (published mirror)
```

Every commit through this repo goes through a safety check (pre-commit hook + GitHub Action + branch protection on master) that blocks personal info and credentials from leaking. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full setup.

## License

MIT. Use them however you want.
