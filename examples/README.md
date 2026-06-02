# Example: capturing a correction, then proposing a lesson

A short, ordinary task showing the loop: the user clarifies a requirement mid-task, the worker
records it (so it isn't forgotten), and at wrap-up proposes a lesson — which stays *pending* until
a human promotes it.

## Layout after install

```
control/                      🔒 read-only to the worker
  rules.md                    e.g. "run names must encode branch identity"
  approved_lessons/index.yaml (reviewed lessons; empty at start)
state/tasks/t1/               ✍️ worker-writable
  run_state.yaml              checkpoint + active_constraints (belief, not truth)
  correction_journal.md       clarified requirements + drift notes
  candidate_lessons.md        pending, un-approved lesson proposals
```

## Walkthrough

```
1. start:     resume-context reads control/rules.md + approved_lessons/index.yaml
              → active_constraints = ["run names must encode branch identity", ...]
2. mid-task:  user says "actually, every chart title must include the run name too"
              → correction-capture appends to correction_journal.md AND adds it to
                run_state.active_constraints  → it's honored for the rest of the task
3. drift:     a check shows the export is missing the month column; user confirms the miss
              → correction-capture records: expected / actual / likely cause / prevention
4. tries to self-authorize: worker decides a rule is inconvenient, edits control/rules.md
              → REJECTED (control/ is a read-only mount)
5. wrap-up:   lesson-propose turns the journal into candidate_lessons.md, tier-tagged
              (working-only / approved-candidate / policy / gate-candidate) — proposals only
6. human:     you review candidate_lessons.md and `git commit` the keepers into
              control/approved_lessons/  → only NOW are they authority
```

## What this proves

- **Corrections survive**: a mid-task clarification goes into active constraints, so it isn't dropped three steps (or one compaction) later.
- **Drift is recorded with a cause**, so the next run has a prevention to follow — not just a one-off fix.
- **`control/` is a trust boundary**: the worker reads it, obeys it, and **cannot rewrite its own authority** (read-only mount).
- **No lesson laundering**: a self-serving "lesson" stays a *proposal* until a human promotes it in git; it can't silently become authority.
- All of it is plain files you `git diff` — no vector/graph infra, no backend.

> `agent-lessonbook` captures drift when something exposes it (a user, a check, a concrete self-observed error). It does not guarantee the agent notices its own drift unprompted.
