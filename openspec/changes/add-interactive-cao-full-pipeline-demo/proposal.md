## Why

Current CAO demo scripts are optimized for one-shot verification and teardown, which makes them hard to use as a reliable interactive test loop. We need a repeatable full-pipeline demo that keeps the tmux-backed session alive for live inspection while operators drive multiple turns through `send-prompt`.

## What Changes

- Add a new interactive CAO demo flow under `scripts/demo/` that launches a CAO-backed agent session (Claude Code with a role prompt) and intentionally keeps the session running until explicitly stopped.
- Capture and print stable session metadata (session manifest path, tmux session/window target, terminal id/log path, workspace paths) so a user can attach to tmux and monitor output live.
- Provide a turn-driving interface for repeated `send-prompt` calls against the same running session, including deterministic prompts for smoke validation.
- Add explicit lifecycle commands for start, send turn, inspect, and stop so the flow is reliable and repeatable in local development.
- Add verification/reporting that confirms the interactive pipeline produced non-empty responses and preserved a single session across multiple turns.

## Capabilities

### New Capabilities
- `cao-interactive-full-pipeline-demo`: Reliable, repeatable demo workflow for CAO-backed interactive sessions that remain alive for tmux inspection while prompts are sent turn-by-turn.

### Modified Capabilities
- None.

## Impact

- Affected code: new or updated demo scripts under `scripts/demo/`, plus helper verification/report logic.
- Affected runtime surfaces: `brain_launch_runtime start-session`, `send-prompt`, and `stop-session` usage patterns in demos.
- Dependencies/systems: local tmux, local/managed CAO server launcher flow, and Claude credential profile for CAO-backed runs.
