# Review only — one-shot adaptive multi-lens orchestrator

<task>Review Only: Adversarial multi-lens review of a PR diff. Classifies the diff, applies the relevant adversarial lenses (pr-review + up to 5 specialized lenses), dedupes findings across lenses, and produces one unified P0/P1/P2 list. One-shot review — does NOT fix anything. For a loop that reviews→fixes→re-reviews until shippable, use ::RF instead. For trivial diffs, classification skips most lenses and the output is essentially a focused pr-review; for substantial diffs, you get the full battery.</task>

<analysis>
STAGE 1 — CLASSIFY THE DIFF (always)
Run `git diff main...HEAD` (or master—check which base). For each lens, decide APPLY or SKIP based on what's in the diff. Output a classification table.

Lens preconditions:
- pr-review: ALWAYS (every diff gets a structured pass against project patterns)
- security-gate: APPLY if diff touches inputs/outputs, auth, network, secrets, dependencies, or any user-controllable surface. SKIP if purely formatting/comments/internal-types/test-fixtures.
- performance-profiler: APPLY if diff touches per-request paths, per-user-action paths, DB queries, network calls, or hot loops. SKIP if build-config/docs/non-runtime.
- ux-critique: APPLY if any change is user-facing—UI, copy, navigation, error states. SKIP if backend-only or developer-tool-only.
- migration-safety: APPLY if any change touches DB schema, API contracts, config formats, or wire protocols. SKIP if no contracts change.
- quality-hunt: APPLY if the diff contains a bug fix. SKIP if purely additive feature work or mechanical refactor.
- hygiene: ALWAYS when the diff contains code files (temp-doc check SKIPs for prose-only diffs). Finds artifacts the diff added — see HYGIENE LENS below.

Output:
| Lens | APPLY/SKIP | Reason |
| pr-review | APPLY | always |
| security-gate | … | … |
| performance-profiler | … | … |
| ux-critique | … | … |
| migration-safety | … | … |
| quality-hunt | … | … |
| hygiene | … | … |

STAGE 2 — APPLY LENSES (parallel if you have Task() in Claude Code; else sequential)
For each APPLY lens, run its standard analysis on the diff. Required output per lens: P0/P1/P2 findings with confidence (1-10) and specific fixes. Standalone deep-dive lenses — ::QS (security review) / ::QP (performance profile) / ::UX (UX critique) / ::QM (migration safety) / ::QH (quality hunt) — have the full lens spec; fall back to those for max rigor. (If you only received this prompt with no access to those trigger definitions, run the named lens from first principles — the parenthetical names tell you what each one does.) The pr-review lens (project-pattern pass: lessons-learned, file categories, coverage gaps) lives only here; no standalone.

Distinguishing rules the standalones may not emphasize:
- Security: try concrete payloads, not abstract category checklists
- Performance: triage dimensions that matter for THESE changes; don't list-check
- Migration: surface the point-of-no-return in the rollback path
- Quality hunt: only fires on bug fixes; goal is finding the same root pattern elsewhere
- HYGIENE LENS (artifacts the diff added): scope = only added lines/files; comment/debug rules apply to code files only, never to .md prose or string literals (quoted text is data); evals/ fixtures/ testdata/ __snapshots__/ exempt; pre-existing content never edited; untracked files in scope only if the request includes the working tree (else a P2 note with the removal command). Tier A (pattern-anchored, fixed by ::RF): P0 = debuggers (debugger;, breakpoint(), pdb.set_trace(), binding.pry, byebug, dbg!) and focused tests in test files only, as test-framework calls (it.only(, describe.only(, test.only(, or bare fit(/fdescribe( — never a method like QuerySet.only( or model.fit(); P1 = scratch files nothing references (*.orig, *.rej, *.bak, *~), comments naming the process (review cycle, per review, P0/P1 fix, TODO(claude)), an unconditional print(/console.log(/println!(/fmt.Println( whose message starts with a debug tag (DEBUG, XXX, HERE, >>>) and isn't behind a verbose/debug flag, and high-confidence temp docs: an added .md whose whole basename is a temp signal (^(notes|scratch|wip|tmp|debug)([-_.].*)?\.md$, any case — NOTES.md yes, RELEASE_NOTES.md/debugging.md no), referenced by nothing, not protected. Tier B (judgment, P2, suggest only, never fixed by the loop): narration comments, commented-out code, logger calls and shell echo, prints without a debug tag or behind a verbose flag (may be real output), tests skipped without a reason, other added unprotected .md. PROTECTED DOCS, never removed: plans (plans/, .planning/, PLAN.md, any .md with an "Acceptance criteria" heading, files named in plan-baseline:/critique cycle commits); functional .md (SKILL.md, AGENTS.md, CLAUDE.md, anything under skills/ agents/ commands/ prompts/ templates/ evals/ tests/ fixtures/ .claude/ .codex/ .agents/ .github/); records (README*, CHANGELOG*, RELEASE*, adr/, decisions/, research/); any doc the plan or PR description names as a deliverable. Removal is recoverable: tracked → git rm in the cycle commit; in-scope untracked → move to ${TMPDIR:-/tmp}/pr-review-quarantine/<repo>-<timestamp>/ and report the path, never rm. Before removing a temp doc, append what fits a canonical doc and is still true of the code, citing where (README*, docs named in CLAUDE.md, or .md under docs/ on the target branch; add lines only), drop the rest. Removal only in fix mode (::RF) on a branch-scoped review; ::RO only reports; a "phase checkpoint" reports temp docs as P2.

STAGE 3 — DEDUPE FINDINGS (mandatory)
Key by (file, line, semantic-fingerprint). Merge: provenance tag (e.g. "security + performance"), highest severity wins. Conflicting fixes → surface BOTH; don't silently pick.

STAGE 4 — CROSS-LENS PATTERNS (0-3; skip with reason if none)
Patterns visible only across lenses: clustering in one file (high-risk area), same root cause as multiple symptoms (fix cause, not instances), lenses disagreeing on severity (surface, don't pick), shared data-flow concerns (riskiest part of diff).

STAGE 5 — VERIFIER LOOP (max 3, confidence-gated)
"What did the lenses miss given this consolidated view?" Output: (a) new findings, (b) confidence 1-10 that the review is now exhaustive.
Stop when: confidence ≥ 8, OR confidence < 8 but no new findings (diminishing returns), OR 3 iterations reached.
Confidence is honest judgment, not a cargo-cult 9.

STAGE 6 — PROOF-OF-UNDERSTANDING FOR EVERY P0
In your own words (not copy-pasted): (1) what the issue is, (2) how it manifests as a concrete failure scenario, (3) what the structural fix is. If you can't produce all three coherently, downgrade to P1 with note "needs human review."
</analysis>

<report>
Output in order:
1. Classification table
2. Verifier loop trace (iterations, confidence per pass, reason for stopping)
3. Findings P0/P1/P2 — each: file:line, issue, lens provenance, proof (what/manifests/fix), confidence
4. Cross-lens patterns (or "none")
5. Overall PR readiness 1-10 with confidence anchor; specific blockers if <8
6. What was skipped (each lens + reason — so misclassification can be spotted)
</report>

<rules>
- Confidence anchors: 1-3=guessing, 4-6=informed but unverified, 7-8=verified by reading code, 9-10=verified with test run or external source
- Skip with one-line reason — never fabricate findings to fill a section
- Run lenses fresh — don't let one lens's findings prejudice the next
- DEDUPE is mandatory; concatenating per-lens output without dedup is a failure of this prompt
- Conflicting fixes → surface explicitly; don't silently pick
- Top 10-15 deduped findings max
- If context missing (no diff, unclear scope, mid-refactor), STOP and ask one clarifying question
- For deeper single-lens passes: ::QS (security review) / ::QP (performance profile) / ::UX (UX critique) / ::QM (migration safety) / ::QH (quality hunt) — names tell you the lens if the trigger isn't available
</rules>
