# Adaptive plan critique (1 cycle)

Critique your plan with 1 adaptive review cycle. The cycle has two phases: FRAME decides what to critique, EXECUTE does the critiquing. Derive the critique dimensions for THIS plan rather than running a fixed checklist.

PHASE A — FRAME (spawn a SEPARATE agent so dimension-selection isn't anchored on the plan author's framing). It returns a critique frame:
1. CLASSIFY: One sentence—user-facing, backend-only, infra, or mixed; and its blast radius.
2. DERIVE DIMENSIONS: 3-6 plan-specific dimensions. Each gets a one-line "why it matters for this plan" + a severity-if-violated (P0/P1/P2). Pull from the catalog as relevant—architecture/consistency, performance/scale, cost, observability/ops, migration & rollout ordering, concurrency/idempotency, data integrity, JTBD, UX—and invent dimensions the catalog misses.
3. ADVERSARIAL FRAMING: derive dimensions by answering "I'm writing the post-mortem for why this plan failed in production six months from now—which dimensions expose that?" Not the neutral "what could we check."
4. MANDATORY EXCLUSIONS: name 1-2 dimensions you are deliberately NOT critiquing, each with a one-line reason. Blocks cherry-picking the easy lenses.

PHASE B — EXECUTE (run the floor first, then every derived dimension):
1. FLOOR (always-on, thin):
   - Assumptions + grounding: list every assumption, rate confidence 1-10. For each <7/10, read the actual code to verify or refute—spawn an agent per assumption group, no agent limit. Don't guess.
   - Validation criteria (per ::QV): per change, observable "done" with edge cases / regression checks / integration contracts / rollback signal. Group P0/P1/P2, mark each automated/manual/production-only. Rate completeness (1-10).
   - Security: ONLY if the classification flags a trust boundary (user input, auth, PHI/health, payment, external API). Else skip with one-line reason.
2. DERIVED DIMENSIONS: critique against each dimension from Phase A. For user-facing dimensions, JTBD (per ::J) and UX (per ::UX) are the tools. Findings carry the severity assigned in the frame. Each proposed fix lists the assumptions it introduces, rated 1-10; ground any <7 (read code/docs, run it) before REWRITE. Between alternative fixes, prefer the one whose assumptions are all cited.
3. REWRITE: Update the operative plan in place: reconcile findings with phases, scope, estimates, decisions, and acceptance criteria. Preserve valid criteria and add or revise affected criteria per ::QV. Record a concise cycle note with evidence, resolved findings, and remaining unknowns. Mark superseded decisions explicitly; do not leave contradictory instructions active or append duplicate acceptance lists. Preserve unrelated user-authored content and meaningful decision history. ADMISSION GATE: a factual claim (existing behavior, numbers, limits, external facts) enters the plan body (instructions, decisions, acceptance criteria) only with a citation—file:line, or a command + its output; a criterion that restates the plan's goal or scope cites the plan and is not parked. Uncited → an "Open questions" section tagged [HYPOTHESIS] + what would verify it; business/preference facts → questions for the user. The cycle note lists the claims this cycle introduced with citations, plus "Open questions: N (+added / −resolved)". Parking isn't deferring: a cited finding and its fix direction stay in the plan, with the pending value named (e.g. "cache rates (TTL: pending Q1)").

RULES:
- Confidence anchors: 1-3=guessing, 4-6=informed but unverified, 7-8=verified by reading code, 9-10=verified with test run or external source
- The FRAME agent picks dimensions but does NOT read all the code—grounding happens in Phase B. Frame fast, execute deep.
- If you cannot determine something from available context (unclear scope, missing code, ambiguous requirement), STOP and ask one clarifying question rather than guessing. An unknown that doesn't block this cycle's dimensions goes to Open questions instead, and the cycle continues
- If a floor lens or derived dimension doesn't apply, say "skipped: [one-line reason]"—never fabricate findings
- NO implementation—planning only
- NO quick wins—optimize for long-term architecture growth
- NO deferring important changes (consistency, performance, correctness)
- Summarize remaining unknowns and recommended next steps
