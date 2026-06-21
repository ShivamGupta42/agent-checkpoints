# Ship cleanup (no CI waiting, approved lessons, clean worktree)

<task>Ship and cleanup. Get the current branch shippable, then squash-merge and remove the feature worktree safely. Prefer doing mechanical prep over stopping: commit dirty work, push missing remotes, open/update the PR. Do not wait on CI; inspect it once and either merge, enable auto-merge, or stop with the concrete pending blocker. Hard-stop only on true merge blockers.</task>

<workflow>
1. Orient: identify repo root, current branch/worktree, base branch (master/main), matching PR if any, and dirty/untracked changes.
2. Make shippable: if dirty, review diff, commit coherent work, push. If no remote branch or PR exists, push and open/update the PR. If a non-trivial behavior change obviously drifts canonical docs (CLAUDE.md, AGENTS.md, README, docs/, runbooks), update docs before final merge and commit/push them.
3. Lessons check: reflect on reusable lessons from this branch with >=7/10 confidence. If any lesson should change future agent/project behavior, ask the user before writing it down. With approval, update the relevant canonical docs (AGENTS.md, CLAUDE.md, README, docs/, runbooks), commit, and push before merge. If the user declines or no lesson qualifies, record that in the report.
4. Merge gate: confirm PR exists, target branch is correct, approval/authorization is satisfied, no conflicts, and platform merge is allowed. Inspect CI/checks once only. Do not poll or wait for pending checks. If required checks are already green or not required, merge. If checks are pending and auto-merge is available/authorized, enable it and stop before cleanup. If checks are failing/red, required approval is missing, conflicts exist, or the platform blocks merge, stop with the next concrete action.
5. Merge + cleanup: squash-merge the PR. Then cd to the master/main worktree BEFORE cleanup. From there remove the feature worktree, delete local+remote branch, git fetch --prune, and git pull. Chain it so nothing runs from the dying worktree.
6. Final check: base worktree clean/current, feature worktree gone, branch gone local+remote, docs/lessons done or intentionally skipped.
</workflow>

<rules>
- Old ::W behavior is the happy path: squash merge, cd to base worktree, remove worktree, delete branch, prune, pull.
- Dirty worktree / missing remote / missing PR are prep tasks, not pre-flight failures.
- Do not wait for CI checks to finish. Inspect once; merge if allowed, enable auto-merge if pending checks are the only blocker and auto-merge is authorized, otherwise stop.
- Do not merge known red/failing required checks, conflicts, wrong base, missing required approval, or uncertain irreversible cleanup.
- Lessons are optional and brief; capture only reusable lessons with >=7/10 confidence, never padding, and never update AGENTS.md/CLAUDE.md/README/docs for lessons without asking the user first.
- Ask one concise question only when blocked by a decision you cannot infer.
</rules>

<report>
Keep output short:
1. Prep done (commits/push/PR/docs) or skipped
2. Lessons: none / prompted+updated / user declined / pending approval
3. Merge gate PASS/FAIL/AUTO-MERGE with reason; include CI status without waiting
4. Cleanup done or blocked
5. Final state and next action
</report>
