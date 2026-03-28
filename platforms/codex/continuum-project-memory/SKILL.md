---
name: continuum-project-memory
description: Create, update, and resume durable project memory under .agent-memory/ with support for init, create_task, spawn_subagent, checkpoint, and continue. Use when AI work needs recoverable project state across thread loss, context resets, subagent handoffs, or switching tools.
---

# Continuum

Use this skill when the user wants durable project state that survives thread loss, context reset, subagent failure, or switching to another AI tool.

## Purpose

Move execution from chat-dependent continuity to state-based continuity.

The source of truth lives in `.agent-memory/`, not in thread history.

## Core Rules

1. Do not treat thread history as the durable source of truth.
2. Keep project memory small, explicit, and recoverable.
3. Always maintain one concrete next step.
4. Store durable facts only:
   - goal
   - constraints
   - current stable state
   - next step
   - key files
5. Do not store speculative reasoning as facts.
6. Prefer updating memory files over writing long conversational recaps.

## Runtime Files

```text
.agent-memory/
  PROJECT.md
  RECOVER.md
  STATE.json
  CHECKPOINTS.jsonl
  DECISIONS.md
  TASKS.md
  tasks/
    task-template.md
    TASK-001.md
  subagents/
    subagent-template.md
    <subagent>.md
```

`PROJECT.md`, `RECOVER.md`, `STATE.json`, and `CHECKPOINTS.jsonl` are the required core.

`DECISIONS.md`, `TASKS.md`, task files, and subagent files are optional scoped extensions. They should be created by the relevant action when needed rather than pre-created by unrelated work.

When scoped files are in use, the canonical pointers are:

- `STATE.json.current_task_file`
- `STATE.json.current_subagent_file`
- `PROJECT.md` current task file section
- `PROJECT.md` current subagent file section
- `RECOVER.md` task file section
- `RECOVER.md` subagent file section

These pointers should be kept in sync whenever the active task or subagent changes.

## Reference Templates

Use these bundled references as the preferred source when creating runtime files:

- [references/project-template.md](./references/project-template.md)
- [references/recover-template.md](./references/recover-template.md)
- [references/state-template.json](./references/state-template.json)
- [references/checkpoints-template.jsonl](./references/checkpoints-template.jsonl)
- [references/decisions-template.md](./references/decisions-template.md)
- [references/tasks-template.md](./references/tasks-template.md)
- [references/task-template.md](./references/task-template.md)
- [references/subagent-template.md](./references/subagent-template.md)

If a reference file is unavailable, create the runtime files directly using the field requirements in this skill.

Optional task and subagent files may be created when the work benefits from more scoped durable state. If they exist, they should stay aligned with `PROJECT.md`, `RECOVER.md`, `STATE.json`, and `CHECKPOINTS.jsonl`.

## Init

Use this when:

- a new project starts
- an existing project adopts this system for the first time

### Required actions

1. Create `.agent-memory/` if it does not exist.
2. Create or update:
   - `.agent-memory/PROJECT.md`
   - `.agent-memory/RECOVER.md`
   - `.agent-memory/STATE.json`
   - `.agent-memory/CHECKPOINTS.jsonl`
   - `.agent-memory/DECISIONS.md`
   - `.agent-memory/TASKS.md`
   - `.agent-memory/subagents/subagent-template.md`
   - `.agent-memory/tasks/task-template.md`
3. Infer from the repository and user request:
   - project goal
   - key constraints
   - one immediate next step
4. Write concise and durable state only.
5. Define:
   - project objective
   - success criteria if they are clear from the repository or request
   - constraints
   - immediate next step
6. Set current status to active and write the first stable checkpoint.

### Return

- summary
- files created
- recommended first task

### Success condition

A fresh AI thread should be able to read `RECOVER.md` and immediately know what to do next.

## Create Task

Use this when:

- a scoped work unit should become explicit
- the project needs a canonical current task file
- checkpoint and continue should track work at task level

### Required actions

1. Define:
   - task id
   - title
   - goal
   - scope
   - owner if known
   - next step
2. Create `.agent-memory/tasks/{task-id}.md`.
3. Update `.agent-memory/TASKS.md`.
4. Update the canonical task pointers in:
   - `.agent-memory/PROJECT.md`
   - `.agent-memory/RECOVER.md`
   - `.agent-memory/STATE.json`
5. Keep the task description concise and durable.

### Return

- task summary
- files created or updated
- next action

## Spawn Subagent

Use this when:

- a task needs to be split into a scoped worker
- a new thread or AI should handle only one narrow part of the work
- a recoverable subagent handoff is useful

### Required actions

1. Define:
   - subagent name
   - subagent role or responsibility
   - subagent status
   - mission
   - scope
   - inputs
   - output contract
2. Create `.agent-memory/subagents/{name}.md`.
3. Update the canonical subagent pointers in:
   - `.agent-memory/PROJECT.md`
   - `.agent-memory/RECOVER.md`
   - `.agent-memory/STATE.json`
4. Include only the context relevant to this subagent.
5. Include the exact files this subagent should inspect first.
6. Define one narrow next step.
7. Do not dump full thread history.
8. Record the subagent's enduring role explicitly so `continue` can restore it later without re-deriving its function.
9. Initialize the subagent status to `active` unless there is a clear reason to start paused.

### Return

- subagent summary
- scoped context package
- next action

## Checkpoint

Use this when:

- a meaningful sub-step is complete
- key files changed
- the thread may end
- work may move to another AI or subagent

### Required actions

1. Append a new JSON line to `.agent-memory/CHECKPOINTS.jsonl`.
2. Update `.agent-memory/PROJECT.md`.
3. Update `.agent-memory/RECOVER.md`.
4. Update `.agent-memory/STATE.json`.
5. If a relevant task file exists, update it.
6. If a relevant subagent file exists, update it.
7. If a task index exists and task state changed, update `.agent-memory/TASKS.md`.
8. If a committed decision was made, update `.agent-memory/DECISIONS.md`.
9. Keep current task and current subagent pointers accurate if they changed during the work.
10. Update subagent status when a scoped worker becomes paused, done, or restored for further use.

### Every checkpoint must record

- what changed
- why it matters
- files changed
- current stable state
- immediate next action
- risks or blockers when relevant
- relevant files

### Quality bar

The next AI should not need the old thread to continue.

### Return

- checkpoint id
- summary
- next action

## Continue

Use this when:

- the thread is lost
- a subagent disappears
- the work resumes later
- the work moves to another AI tool

### Read order

Read in this order:

1. `.agent-memory/RECOVER.md`
2. `.agent-memory/PROJECT.md`
3. `.agent-memory/STATE.json`
4. `.agent-memory/TASKS.md` if it exists
5. the current task file from the canonical pointers, if it exists
6. discover available subagent files in `.agent-memory/subagents/` if the directory exists
7. the current subagent file from the canonical pointers, if it exists
8. `.agent-memory/DECISIONS.md` if it exists and current work depends on committed decisions
9. listed key source or document files

### Then produce

- current understanding
- immediate next action
- missing information if any
- first code or document change to make
- files to inspect first
- any conflict between memory and current project state

### Subagent restore rule

If subagent files are present, ask the user one concise restore-scope question:

- restore only the current subagent
- restore specific subagents by name
- restore all subagents

After the user chooses, immediately read the selected subagent files and restore them by reporting:

- role or responsibility
- status
- mission
- current state
- pending next step

For each selected subagent, restore it for use by:

- preserving current project and task context while restoring subagents
- spawning a real agent for each selected subagent
- updating its status to `restored` or `active` as appropriate
- updating the canonical current subagent pointers when the restored subagent becomes the active one
- recording spawned agent identifiers when available
- making the restored next step explicit so work can continue immediately

### Continue rule

Do not ask for the full old thread unless truly blocked.
Trust project memory over conversational history unless the repository clearly contradicts it.
Use the canonical task and subagent pointers to decide which scoped files are relevant.
Do not restore all subagents by default when a narrower restore path is possible.
Treat `done` subagents as historical by default unless the user explicitly chooses to restore them for further use.
Do not drop other tasks or project-level context while restoring subagents.

## Summary

This skill manages durable project state through files.

`init` creates the memory.

`create task` creates scoped durable state for a unit of work.

`spawn subagent` creates scoped durable state for a narrow worker.

`checkpoint` preserves stable progress.

`continue` resumes from durable state.
