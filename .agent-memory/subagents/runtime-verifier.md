# SUBAGENT

## Name

runtime-verifier

## Mission

Verify that Continuum's richer runtime memory model works end-to-end once optional files are instantiated.

## Scope

Only handle:

- validating the task and subagent pointers
- checking that recovery can locate the right scoped files

Do not handle:

- changing the public product scope
- introducing new runtime file categories

## Inputs

- `.agent-memory/RECOVER.md`
- `.agent-memory/PROJECT.md`
- `.agent-memory/STATE.json`
- `.agent-memory/TASKS.md`
- `.agent-memory/tasks/TASK-001.md`
- `platforms/codex/continuum-project-memory/SKILL.md`

## Current State

The richer runtime memory flow has been validated: the recovery pass can locate the active task and subagent through the canonical pointers and resume without thread history.

## Assumptions

- the current active task is `TASK-001`
- the subagent should validate the flow rather than invent new scope

## Files Touched

- `.agent-memory/subagents/runtime-verifier.md`
- `.agent-memory/RECOVER.md`
- `.agent-memory/PROJECT.md`
- `.agent-memory/STATE.json`

## Decisions

- Use a validation-focused subagent instead of a feature-focused one so the first run exercises the recovery model directly.

## Pending

No further work in this subagent. The validation result can now inform the next implementation slice.

## Output Contract

Return:

- summary
- changed files
- risks
- next step
