# DECISIONS

## D-001

Date: 2026-03-28
Decision: Continuum runtime memory should grow through action-triggered file creation rather than by pre-creating every optional file outside the workflow.
Why: This keeps the core lightweight while ensuring optional files only appear when a real action needs them.
Impact: `init`, `create_task`, `spawn_subagent`, `checkpoint`, and `continue` must define when optional files are created, read, and updated.
