## 1. Interactive Demo Scaffold

- [ ] 1.1 Create a new demo pack under `scripts/demo/` for the interactive CAO full-pipeline workflow with README and input fixtures.
- [ ] 1.2 Add startup command wiring that builds the Claude brain manifest, pins CAO startup to `http://127.0.0.1:9889`, and starts a `cao_rest` session with role prompt support plus explicit name-based `--agent-identity`.
- [ ] 1.3 Persist startup outputs to a machine-readable state artifact with primary `agent_identity` plus session manifest, tmux target/session name, terminal id/log path, and workspace paths.
- [ ] 1.4 If a prior demo session is marked active, force-close it by recorded `agent_identity` before launching the replacement session and rewriting state.

## 2. Turn Driving and Inspection

- [ ] 2.1 Implement a turn-driving command that reads state and calls `brain_launch_runtime send-prompt` against the active session by stored `agent_identity`.
- [ ] 2.2 Record per-turn outputs (prompt, response text, timestamps, exit status) and fail when response text is empty.
- [ ] 2.3 Add an inspect command/output mode that prints tmux attach target and terminal log tail command from stored metadata.

## 3. Teardown and Reliability Guardrails

- [ ] 3.1 Implement an explicit stop command that calls `brain_launch_runtime stop-session` by stored `agent_identity` and updates state to inactive.
- [ ] 3.2 Handle stale/missing remote sessions during stop gracefully while still marking local state inactive.
- [ ] 3.3 Reuse CAO launcher ownership guardrails from existing CAO tmux demos while enforcing the fixed loopback CAO target at `http://127.0.0.1:9889`.

## 4. Verification and Documentation

- [ ] 4.1 Add verifier/report generation that asserts one `agent_identity` is reused across at least two turns and responses are non-empty.
- [ ] 4.2 Add unit/integration tests for state-file handling and lifecycle command behavior, including start replacement, name-based targeting, and stop failure modes.
- [ ] 4.3 Document a reproducible operator workflow in README (start, replacement behavior, tmux attach, send turns, inspect logs, stop) using the fixed loopback CAO target.
