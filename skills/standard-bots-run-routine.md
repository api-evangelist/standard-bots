---
name: RO1 run a routine
description: List, play, monitor, pause and stop a routine authored in the Standard Bots Routine Editor, and read/update its runtime variables.
api: RO1 Robotics REST API (/api/v1)
source: Grounded in the official Standard Bots Python SDK (PyPI standardbots 2.20260617.2); every operation below exists verbatim in standardbots.auto_generated.apis.
operations:
  - list                 # GET  /api/v1/routine-editor/routines
  - load                 # GET  /api/v1/routine-editor/routines/{routine_id}
  - play                 # POST /api/v1/routine-editor/routines/{routine_id}/play
  - get_state            # GET  /api/v1/routine-editor/routines/{routine_id}/state
  - pause                # POST /api/v1/routine-editor/routines/{routine_id}/pause
  - stop                 # POST /api/v1/routine-editor/stop
  - get_step_variables   # GET  /api/v1/routine-editor/routines/{routine_id}/step-variables
  - update               # POST /api/v1/routine-editor/variables/{variable_name}
---

# RO1 run a routine

Play and supervise a pre-authored RO1 routine. **A running routine drives physical motion — require human confirmation before `play`.**

## Auth
Control box host, headers `Authorization: Bearer <token>`, `robot-kind: live|simulated`, `Content-Type: application/json`.

## Steps
1. **Find the routine.** `list` (`GET /api/v1/routine-editor/routines`) and pick the target `routine_id`. `load` returns its full definition.
2. **Play.** `play` on `{routine_id}`. Fails with `cannot_play_routine` if the routine is not in a playable state.
3. **Monitor.** Poll `get_state` for `{routine_id}` until it reports completion. Read live values with `get_step_variables` (fails with `routine_must_be_running` if the routine is not running).
4. **Adjust variables (optional).** `update` a `{variable_name}` while the routine runs to change behavior on the fly.
5. **Pause / resume.** `pause` halts a running routine (`routine_must_be_running` otherwise).
6. **Stop.** `stop` (`POST /api/v1/routine-editor/stop`) ends the routine and all ongoing motion.

## Error handling
`{ "error": <code>, "message": <text> }`. Relevant codes: `routine_does_not_exist`, `cannot_play_routine`, `routine_must_be_running`, `routine_must_be_playing`. See `errors/standard-bots-error-codes.yml`.
