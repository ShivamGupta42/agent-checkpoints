# Review-and-fix loop — iterate until no P0/P1 (P2s deferred)

<task>Review-and-Fix Loop: Iterate review→fix→re-review until ZERO P0 and ZERO P1 findings remain. P2 findings are surfaced but NEVER fixed in this loop and never block exit. Use this when the goal is "make the diff shippable", not "make the diff perfect". For a one-shot review with no fix, use ::RO. For a one-shot fix with no re-review gate, use ::F.</task>

<workflow>
Initialize cycle counter N=1 (hard cap: 5 cycles).

PER CYCLE:

1. REVIEW (full ::RO pass on current diff): Classify the diff, apply lenses (pr-review + security/performance/ux/migration/quality-hunt where they APPLY), dedupe findings, run verifier loop, produce proof-of-understanding for every P0. Output the consolidated P0/P1/P2 list. Each cycle's review MUST be a fresh ::RO run on the current diff — do NOT carry forward findings from prior cycles, since the fixes may have introduced new issues or invalidated old ones.

2. EXIT GATE (evaluate immediately after review):
   - If ZERO P0 and ZERO P1 → STOP with status CLEAN. Report remaining P2s as "deferred, not blocking". Done.
   - If cycle N == 5 AND P0/P1 still remain → STOP with status BLOCKED. Report unresolved findings and why automated convergence failed (often: ambiguous fix, conflicting lens recommendations, missing context). Ask the user how to proceed.
   - Otherwise → continue to step 3.

3. FIX (per ::F semantics) — for each P0 and P1 from step 1:
   - State validation criteria (the observable behavior that proves it's fixed)
   - Implement the fix
   - Verify against criteria
   - Check for regressions in adjacent code
   - Fix One Find All: grep the codebase for the same root pattern; fix every occurrence, not just the reported instance
   - EXPLICITLY SKIP all P2 findings. P2s are out of scope for this loop by design.

4. COMMIT: `git add -A && git commit -m "review cycle N: fix P0/P1 [brief]" && git push`

5. LOOP: Increment N. Return to step 1 with the fresh diff.

EXIT REPORT (mandatory on stop):
- Cycles run, exit status (CLEAN / BLOCKED)
- Per-cycle: P0/P1 counts found → fixed (so the convergence trajectory is visible)
- Final remaining P2s, tagged "deferred — not blocking ship"
- Overall ship-readiness (1-10) with confidence anchor
- If BLOCKED: specific reason and one clarifying question
</workflow>

<rules>
- Exit conditions are ONLY: (a) zero P0+P1 remaining, OR (b) 5-cycle cap. No other stop reasons — not "looks good enough", not "the remaining P1 is minor", not "I'd downgrade this".
- P2 findings are NEVER fixed in this loop. They are reported as deferred at exit. If the user wants P2s addressed, they run a follow-up cycle explicitly.
- Confidence anchors: 1-3=guessing, 4-6=informed but unverified, 7-8=verified by reading code, 9-10=verified with test run
- A re-review that surfaces NEW P0/P1 caused by your fixes is a regression — keep looping, those count toward the exit gate just like original findings
- Downgrading a P0 to P1 (or P1 to P2) is allowed ONLY when re-reading the code reveals the lens overstated severity AND you write a one-line justification with confidence ≥7/10. Do not silently re-rank to escape the loop.
- If context is missing (unclear fix, ambiguous scope, conflicting lens recommendations), STOP and ask one clarifying question rather than guessing — that is BLOCKED status, not CLEAN
- Confidence required to exit CLEAN: ≥7/10 that a fresh ::RO pass on the current state would also return zero P0/P1
</rules>
