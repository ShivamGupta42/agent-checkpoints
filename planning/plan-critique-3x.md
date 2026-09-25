# Adaptive plan critique (3 cycles)

Critique your plan with 3 full adaptive review cycles. Each cycle has two phases: FRAME (decide what to critique) then EXECUTE (critique it). Derive the dimensions per plan, not a fixed checklist. Commit and push after each cycle.

PER CYCLE:
PHASE A — FRAME (spawn a SEPARATE agent so dimension-selection isn't anchored on the plan author's framing):
1. CLASSIFY: One sentence—user-facing, backend-only, infra, or mixed; and blast radius. Re-classify per cycle—scope can shift.
2. DERIVE DIMENSIONS: 3-6 plan-specific dimensions, each with a one-line "why it matters" + severity-if-violated (P0/P1/P2). Pull from the catalog (architecture/consistency, performance/scale, cost, observability/ops, migration & rollout ordering, concurrency/idempotency, data integrity, JTBD, UX) and invent what it misses.
3. ADVERSARIAL FRAMING: derive by answering "I'm writing the post-mortem for why this plan failed in prod six months out—which dimensions expose that?"
4. MANDATORY EXCLUSIONS: name 1-2 dimensions you are deliberately NOT critiquing + one-line reason each.
5. CROSS-CYCLE ATTACK (cycles 2-3): open by attacking the previous cycle's frame—"what did the last cycle fail to even look at?" New/promoted dimensions must appear here.

PHASE B — EXECUTE (floor first, then every derived dimension):
1. FLOOR (always-on, thin): Assumptions + grounding (cycles 2+: first ground the claims the previous cycle introduced; then list all, rate 1-10, read code for any <7/10—spawn an agent per group, don't guess); Validation criteria (per ::QV, P0/P1/P2, automated/manual/production-only, rate completeness 1-10); Security ONLY if a trust boundary is in scope, else skip with reason.
2. DERIVED DIMENSIONS: critique against each Phase A dimension. User-facing → JTBD (::J) + UX (::UX). Findings carry the frame's severity. Each proposed fix lists the assumptions it introduces, rated 1-10; ground any <7 (read code/docs, run it) before REWRITE. Between alternative fixes, prefer the one whose assumptions are all cited.
3. REWRITE: Update the operative plan in place: reconcile findings with phases, scope, estimates, decisions, and acceptance criteria. Preserve valid criteria and add or revise affected criteria per ::QV. Record a concise cycle note with evidence, resolved findings, and remaining unknowns. Mark superseded decisions explicitly; do not leave contradictory instructions active or append duplicate acceptance lists. Preserve unrelated user-authored content and meaningful decision history. ADMISSION GATE: a factual claim (existing behavior, numbers, limits, external facts) enters the plan body (instructions, decisions, acceptance criteria) only with a citation—file:line, or a command + its output; a criterion that restates the plan's goal or scope cites the plan and is not parked. Uncited → an "Open questions" section tagged [HYPOTHESIS] + what would verify it; business/preference facts → questions for the user. The cycle note lists the claims this cycle introduced with citations, plus "Open questions: N (+added / −resolved)". Parking isn't deferring: a cited finding and its fix direction stay in the plan, with the pending value named (e.g. "cache rates (TTL: pending Q1)").
4. COMMIT: `git add -A && git commit -m "critique cycle N/3: [key findings]" && git push`

RULES:
- Confidence anchors: 1-3=guessing, 4-6=informed but unverified, 7-8=verified by reading code, 9-10=verified with test run or external source
- The FRAME agent picks dimensions but does NOT read all the code—grounding happens in Phase B. Frame fast, execute deep.
- If you cannot determine something from available context (unclear scope, missing code, ambiguous requirement), STOP and ask one clarifying question rather than guessing. An unknown that doesn't block this cycle's dimensions goes to Open questions instead, and the cycle continues
- If a floor lens or derived dimension doesn't apply, say "skipped: [one-line reason]"—never fabricate findings
- NO implementation—planning only
- NO quick wins—optimize for long-term architecture growth
- NO deferring important changes (consistency, performance, correctness)
- MUST complete all 3 cycles before stopping
- Each cycle must go DEEPER than the last—the cross-cycle attack challenges not just conclusions but the choice of what to examine
- After final cycle: summarize remaining unknowns and recommended next steps
