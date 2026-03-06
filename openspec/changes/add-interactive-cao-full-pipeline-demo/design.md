## Context

Current CAO demo scripts in `scripts/demo/` are primarily one-shot validation flows that launch, prompt once (or in a fixed sequence), verify, and tear down automatically. They are good for CI stability but not for hands-on runtime inspection because operators cannot reliably keep a session alive while iteratively sending prompts and observing tmux output in real time. The requested change introduces a deterministic interactive pipeline for CAO-backed Claude sessions with explicit lifecycle boundaries.

## Goals / Non-Goals

**Goals:**
- Provide a repeatable local interactive demo that starts a CAO-backed Claude session and keeps it alive for manual inspection until the user intentionally stops it.
- Make session state discoverable by emitting stable metadata needed for `tmux attach`, terminal-log tailing, and subsequent `send-prompt` calls.
- Support multi-turn prompt driving against one session identity with simple command surfaces suitable for manual and scripted usage.
- Preserve existing CI-friendly demo behavior by isolating this interactive flow instead of changing current one-shot demos.

**Non-Goals:**
- Replacing all existing demo scripts with interactive behavior.
- Introducing remote/non-local tmux control semantics beyond existing local constraints.
- Defining a new runtime backend or changing CAO protocol contracts.

## Decisions

1. Add a dedicated interactive demo pack instead of modifying existing one-shot demos.
Rationale: Current demos are tuned for deterministic teardown and report verification. Mixing interactive persistence into them would increase complexity and flake risk for established flows.
Alternatives considered:
- Add a `--keep-session` flag to existing demo scripts: lower duplication but increases branching and testing matrix across multiple demos.
- Reuse `cao-claude-esc-interrupt` as-is: close behavior but purpose-built around Esc interruption, not generic turn-loop workflows.

2. Use explicit lifecycle commands (`start`, `send-turn`, `inspect`, `stop`) over one monolithic script.
Rationale: Interactive workflows are operator-driven; explicit subcommands avoid hidden teardown and make repeat runs predictable.
Alternatives considered:
- Single long-running script loop: simpler initial UX but harder to recover/debug after partial failures.

3. Persist session metadata in a workspace-local machine-readable state file.
Rationale: `send-prompt` and inspection steps need stable references (session identity, tmux target, terminal id/log path) across invocations.
Alternatives considered:
- Keep data in process memory only: incompatible with separate command invocations.
- Parse ad-hoc logs each time: fragile and harder to maintain.

4. Keep local-only CAO assumptions aligned with existing tmux-based demos.
Rationale: tmux inspection and key injection patterns are local host operations; enforcing local CAO base URL avoids ambiguous remote behavior.
Alternatives considered:
- Allow remote CAO endpoints: introduces uncertainty around terminal ownership/visibility and would need additional transport guarantees.

## Risks / Trade-offs

- [Risk] Session leaks when users forget explicit stop.
  - Mitigation: Provide `stop` command and clear printed reminders; optionally include a cleanup helper that enumerates stale demo sessions.
- [Risk] Ownership mismatch with untracked local CAO server process.
  - Mitigation: Reuse launcher ownership checks and retry pattern used by current CAO demos.
- [Risk] Interactive timing variability may reduce deterministic assertions.
  - Mitigation: Keep verification focused on invariant outcomes (non-empty responses, same session identity across turns, accessible tmux target) instead of brittle timing conditions.
- [Trade-off] Additional script surface area to maintain.
  - Mitigation: Share helper logic and structure with existing CAO demo patterns where practical.
