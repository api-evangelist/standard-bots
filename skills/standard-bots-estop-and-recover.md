---
name: RO1 emergency stop and recover
description: Engage the RO1 emergency braking system, inspect health and recovery status, and recover the robot from a fault state.
api: RO1 Robotics REST API (/api/v1)
source: Grounded in the official Standard Bots Python SDK (PyPI standardbots 2.20260617.2); every operation below exists verbatim in standardbots.auto_generated.apis.
operations:
  - engage_emergency_stop   # POST /api/v1/movement/brakes/emergency-stop
  - get_health              # GET  /api/v1/status/health
  - get_status              # GET  /api/v1/recovery/status
  - recover                 # POST /api/v1/recovery/recover
  - trigger_user_fault      # POST /api/v1/faults/user-fault
  - get_brakes_state        # GET  /api/v1/movement/brakes
---

# RO1 emergency stop and recover

Immediately stop the RO1 and bring it back to a controllable state. **`engage_emergency_stop` halts physical hardware at once — this is the safe-fail path; the recover step re-enables motion and must be human-confirmed.**

## Auth
Control box host, headers `Authorization: Bearer <token>`, `robot-kind: live|simulated`, `Content-Type: application/json`.

## Steps
1. **Emergency stop.** `engage_emergency_stop` (`POST /api/v1/movement/brakes/emergency-stop`) engages the emergency braking system immediately. Use this the moment anything is wrong.
2. **Assess.** `get_health` for overall robot health and `get_status` (`GET /api/v1/recovery/status`) for recovery state. Optionally `get_brakes_state`.
3. **Recover.** When it is safe to resume, `recover` (`POST /api/v1/recovery/recover`) attempts to clear the fault state. Inspect the response; a failed attempt returns `recovery_error`.
4. **Trigger a user fault (optional).** From within routine logic, `trigger_user_fault` raises a controlled fault to stop execution safely.

## Error handling
`{ "error": <code>, "message": <text> }`. Relevant codes: `recovery_error`, `robot_brakes_engage_failed`, `robot_brakes_disengage_failed`. See `errors/standard-bots-error-codes.yml`. Never auto-`recover` without a human confirming the workspace is clear.
