---
name: RO1 pick-and-place move
description: Safely acquire API control of a Standard Bots RO1, move the arm to a pose, and actuate the gripper to pick and place an object.
api: RO1 Robotics REST API (/api/v1)
source: Grounded in the official Standard Bots Python SDK (PyPI standardbots 2.20260617.2); every operation below exists verbatim in standardbots.auto_generated.apis.
operations:
  - get_configuration_state_control      # GET  /api/v1/status/control-mode
  - set_configuration_control_state      # POST /api/v1/status/control-mode
  - get_health                           # GET  /api/v1/status/health
  - get_brakes_state                     # GET  /api/v1/movement/brakes
  - set_brakes_state                     # POST /api/v1/movement/brakes
  - get_arm_position                     # GET  /api/v1/movement/position/arm
  - set_arm_position_controlled          # POST /api/v1/movement/position/arm/controlled
  - send_heartbeat                       # POST /api/v1/movement/position/arm/controlled/{command_id}/heartbeat
  - control_gripper                      # POST /api/v1/equipment/end-effector/control
  - get_gripper_configuration            # GET  /api/v1/equipment/end-effector/configuration
---

# RO1 pick-and-place move

Use this skill to move a Standard Bots RO1 arm to a target pose and pick/place an object. **This commands physical motion — treat every write step as safety-critical and require human confirmation before executing.**

## Auth
All requests go to the robot control box (`http://<robot-control-box>:3000`) with headers:
- `Authorization: Bearer <developer-api-token>`
- `robot-kind: live` (use `simulated` to rehearse without moving hardware)
- `Content-Type: application/json`

## Steps
1. **Confirm health.** `get_health` (`GET /api/v1/status/health`). Do not proceed if the robot is faulted — run the recovery skill first.
2. **Acquire API control.** Check `get_configuration_state_control`, then `set_configuration_control_state` to put the robot under API control. Write moves fail with `api_control_required` otherwise.
3. **Disengage brakes.** `get_brakes_state`; if engaged, `set_brakes_state` to disengage. Moving with brakes engaged fails with `brakes_must_be_disengaged`; the robot must also be idle (`robot_not_idle`).
4. **Read current pose (optional).** `get_arm_position` to plan the approach.
5. **Move to the approach pose.** `set_arm_position_controlled` — this returns a `command_id`. Immediately begin a heartbeat loop: repeatedly `send_heartbeat` for `{command_id}` until the move completes, or the motion is deauthorized (the command-lease pattern; there is no idempotency key).
6. **Actuate the gripper to pick.** `get_gripper_configuration` to confirm the end-effector, then `control_gripper` with a grip command.
7. **Move to the place pose** with another `set_arm_position_controlled` + heartbeat loop.
8. **Release.** `control_gripper` with an open/move command to place the object.

## Error handling
Errors return `{ "error": <code>, "message": <text> }`. Common codes: `api_control_required`, `robot_not_idle`, `brakes_must_be_disengaged`, `robot_out_of_compliance_bounds`. See `errors/standard-bots-error-codes.yml`. On any fault, stop issuing motion and switch to recovery.
