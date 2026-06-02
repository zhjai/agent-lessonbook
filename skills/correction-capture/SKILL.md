---
name: correction-capture
description: Use the moment the user corrects or clarifies a requirement mid-task ("actually…", "that's not what I meant", "you skipped X again"), OR when drift is exposed (a check fails, the user points out a miss, or you concretely observe your own earlier error). Record it to the task's correction journal and add it to the run's active constraints so it isn't forgotten or repeated. Do not use for routine progress notes. Captures corrections/drift; it does not by itself make the agent notice drift.
license: MIT
metadata:
  version: "0.2.0"
  author: zhjai
  tags: "lessonbook, corrections, drift, requirements, journal"
  related_skills: "resume-context, lesson-propose"
---

# Correction Capture

A clarified requirement and an exposed drift are usually the same moment ("you skipped the
baseline again" is both). One journal, one skill. The worker records; it does not gain authority.

## When to use
- The user corrects or clarifies mid-task: "actually…", "not like that", "from now on in this project…", "don't do X again".
- Drift is **exposed**: a check/test fails, the user points out a missed detail, or you concretely observe that an earlier step went the wrong way.

> This skill captures drift when something *exposes* it. It does not guarantee the agent reliably notices its own drift on its own.

## Procedure
1. Append an entry to `state/tasks/<task_id>/correction_journal.md`:
   ```md
   ## <date> — correction | drift
   Trigger: <what surfaced it: user said X / check Y failed / self-observed>
   Expected: <what should have happened>
   Actual:   <what happened>
   Likely cause: <why — e.g. constraint wasn't in active_constraints; optimized for tests not user-visible output>
   Prevention: <concrete check or step to avoid repeating it>
   Scope: task-only | candidate for future tasks
   ```
2. If it's an active requirement for the rest of this task, add it to `run_state.yaml`'s `active_constraints` immediately — so it's honored three steps later, not forgotten.
3. If `Scope` is "candidate for future tasks", flag it so `lesson-propose` picks it up at wrap-up. **Do not** promote it yourself.

## Outputs
- Appended `correction_journal.md` entry.
- Updated `run_state.yaml` `active_constraints` (when it applies to the rest of the task).

## Do not
- Do not write to `control/` — corrections are worker observations, not authority, until a human reviews them.
- Do not silently "absorb" a correction without recording it — recording is the point (so it survives compaction and the next session).
- Do not over-capture: routine progress is not a correction. Capture when expectation and reality diverged.
