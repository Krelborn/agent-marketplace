# Smoke-gate Brief Template

The orchestrator dispatches this brief to a Coder subagent (`subagent_type: "tsc-react-dev-plugin:coder"`) immediately after each unit's primary implementation. Substitute the changed-files list and touched-test-files list. Do not modify the brief otherwise.

---

**You are the smoke gate for this unit. Do not write new code unless fixing a smoke failure.**

Files this unit changed: `[list]`. Test files this unit touched or wrote: `[list]`.

1. Run the project's typecheck (`yarn typecheck` or equivalent). Scope to changed files if supported; otherwise run the project-wide typecheck.
2. Run only the test files listed above. **Do not run the full suite** — that happens later in the Verify phase.
3. If both pass, report `smoke-gate: pass` and stop.
4. If either fails, fix only the specific failure (typecheck error or failing test). Do not refactor unrelated code, do not add features, do not address style/design/"looks incomplete" concerns — those belong to the end-of-implementation review.
5. Re-run only the failing check after each fix. Cap at **2 fix attempts on the same failure**. After the second failed attempt, stop and report `smoke-gate: fail` with the unresolved failure output.
6. Final report must be exactly one of `smoke-gate: pass` or `smoke-gate: fail: [reason + key output]`.
