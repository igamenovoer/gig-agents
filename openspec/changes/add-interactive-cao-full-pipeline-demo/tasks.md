## 1. Interactive Demo Scaffold

- [ ] 1.1 Create a new demo pack under `scripts/demo/` for the interactive CAO full-pipeline workflow with README and input fixtures.
- [ ] 1.2 Add startup command wiring that builds the Claude brain manifest and starts a `cao_rest` session with role prompt support.
- [ ] 1.3 Persist startup outputs to a machine-readable state artifact (session manifest, tmux target/session name, terminal id/log path, workspace paths).

## 2. Turn Driving and Inspection

- [ ] 2.1 Implement a turn-driving command that reads state and calls `brain_launch_runtime send-prompt` against the active session.
- [ ] 2.2 Record per-turn outputs (prompt, response text, timestamps, exit status) and fail when response text is empty.
- [ ] 2.3 Add an inspect command/output mode that prints tmux attach target and terminal log tail command from stored metadata.

## 3. Teardown and Reliability Guardrails

- [ ] 3.1 Implement an explicit stop command that calls `brain_launch_runtime stop-session` and updates state to inactive.
- [ ] 3.2 Handle stale/missing remote sessions during stop gracefully while still marking local state inactive.
- [ ] 3.3 Reuse CAO launcher ownership and local-only guardrails from existing CAO tmux demos for consistent behavior.

## 4. Verification and Documentation

- [ ] 4.1 Add verifier/report generation that asserts one session identity is reused across at least two turns and responses are non-empty.
- [ ] 4.2 Add unit/integration tests for state-file handling and lifecycle command behavior (start/send/stop failure modes).
- [ ] 4.3 Document a reproducible operator workflow in README (start, tmux attach, send turns, inspect logs, stop).
