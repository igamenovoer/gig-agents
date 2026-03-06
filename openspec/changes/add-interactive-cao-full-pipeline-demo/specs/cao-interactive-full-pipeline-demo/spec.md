## ADDED Requirements

### Requirement: Interactive demo session startup
The demo workflow SHALL provide a startup command that builds a Claude CAO runtime context, starts a `cao_rest` session with a role prompt, and writes a state artifact containing the session identity and inspection metadata.

#### Scenario: Startup succeeds with local prerequisites
- **WHEN** the user runs the interactive startup command with valid local CAO availability and credentials
- **THEN** the command produces a state artifact containing `session_manifest`, `session_name`, `terminal_id`, and `terminal_log_path`
- **AND** the command exits without calling `stop-session`

#### Scenario: Startup fails safely when prerequisites are missing
- **WHEN** required runtime prerequisites (for example tmux, credentials, or CAO connectivity) are unavailable
- **THEN** the command exits with an explicit reason and does not create a partial interactive state marked as active

### Requirement: Multi-turn prompt driving against a live session
The demo workflow SHALL provide a turn-driving command that reads the persisted state artifact and sends a prompt through `brain_launch_runtime send-prompt` to the same active session.

#### Scenario: Sequential prompts use one session identity
- **WHEN** the user runs the turn-driving command multiple times after a successful startup
- **THEN** each turn targets the same `session_manifest` recorded by startup
- **AND** each turn records a non-empty response in per-turn output artifacts

#### Scenario: Turn-driving rejects missing or inactive session state
- **WHEN** the user runs the turn-driving command before startup or after stop
- **THEN** the command fails with a clear actionable message indicating that no active interactive session exists

### Requirement: Live inspection affordances
The demo workflow SHALL expose enough metadata for live tmux and log inspection while the session remains active.

#### Scenario: User can attach and observe
- **WHEN** startup completes
- **THEN** the workflow outputs or stores a tmux attach target derived from session metadata
- **AND** the workflow outputs or stores a terminal log path suitable for tailing CAO output

### Requirement: Explicit interactive teardown
The demo workflow SHALL provide an explicit stop command that terminates the active interactive session and marks the state as no longer active.

#### Scenario: Stop cleans up active session
- **WHEN** the user runs the stop command with an active state artifact
- **THEN** the command calls `brain_launch_runtime stop-session` for the recorded session identity
- **AND** subsequent turn-driving attempts fail until startup is run again

#### Scenario: Stop is safe when session is already gone
- **WHEN** the recorded session no longer exists remotely
- **THEN** the command exits gracefully and updates local state to inactive
