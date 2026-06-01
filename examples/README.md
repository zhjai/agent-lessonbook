# Example: governance memory in a short task (standalone)

`agent-memory` is useful **without** any completion gate. This walkthrough shows the permission boundary doing its job in an ordinary task — no long-task machinery.

## Layout after install

```
control/                      🔒 read-only to the worker
  rules.md                    e.g. "run names must encode branch identity"
  approved_lessons/index.yaml (reviewed lessons; empty at start)
state/tasks/t1/               ✍️ worker-writable
  run_state.yaml              checkpoint (belief, not truth)
  decision_log.md             decisions / risks
  candidate_lessons.md        pending, un-approved lessons
```

## Walkthrough

```
1. start:    resume-context reads control/rules.md + approved_lessons/index.yaml
             → active_constraints = ["run names must encode branch identity", ...]
2. work:     worker appends to state/tasks/t1/decision_log.md as it goes
3. mid-task: worker decides a rule is inconvenient, tries to edit control/rules.md
             → REJECTED (control/ is a read-only mount)
4. learning: worker writes "skip the case check next time" to candidate_lessons.md
             → stays PENDING; it is NOT in approved_lessons/, so it never acts as authority
5. wrap-up:  lesson-promote tags the candidate (working-only / approved-candidate / policy / gate)
             and writes a promotion note for human/CI review — the worker cannot self-approve
```

## What this proves

- **`control/` is a trust boundary**: the worker reads it, obeys it, and **cannot rewrite its own authority** (filesystem read-only mount).
- **`state/` is a scratchpad**, not truth — `run_state` says what the agent *believes* it did.
- **No lesson laundering**: a self-serving "lesson" stays a *proposal* until reviewed; it can't silently weaken future behavior.
- You get cross-task continuity (rules, decisions, lessons) **without** vector/graph infra and **without** any completion gate.

For long / high-stakes tasks, add [`agent-completion-gate`](https://github.com/zhjai/agent-completion-gate) on top.
