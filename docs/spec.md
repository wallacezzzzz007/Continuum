# Continuum Specification

## Purpose

Continuum is a portable state layer for AI work.

It is designed to keep execution recoverable across:

- thread loss
- context reset
- subagent failure
- switching to another AI tool

The source of truth lives in files, not in thread history.

## Core Actions

### `init`

Use this when:

- a new project starts
- an existing project adopts this system for the first time

Expected result:

- `.agent-memory/` is created
- the base memory files exist
- optional scaffold files for decisions, tasks, and scoped work exist
- the project goal, constraints, and next step are recorded
- the first stable checkpoint exists

Create or update:

- `.agent-memory/PROJECT.md`
- `.agent-memory/RECOVER.md`
- `.agent-memory/STATE.json`
- `.agent-memory/CHECKPOINTS.jsonl`
- `.agent-memory/DECISIONS.md`
- `.agent-memory/TASKS.md`
- `.agent-memory/subagents/subagent-template.md`
- `.agent-memory/tasks/task-template.md`

Return:

- summary
- files created
- recommended first task

### `create_task`

Use this when:

- a scoped work unit should become explicit
- the project needs a canonical current task

Expected result:

- a task file exists
- the task index is updated
- current-task pointers are updated consistently

Create or update:

- `.agent-memory/tasks/{task-id}.md`
- `.agent-memory/TASKS.md`
- `.agent-memory/PROJECT.md`
- `.agent-memory/RECOVER.md`
- `.agent-memory/STATE.json`

Return:

- task summary
- files created or updated
- next action

### `spawn_subagent`

Use this when:

- a narrow scoped worker is needed
- a new AI thread should handle only one part of the work

Expected result:

- a scoped subagent memory file exists
- the subagent has minimal relevant context
- the subagent has one narrow next step
- current-subagent pointers are updated consistently

Create:

- `.agent-memory/subagents/{name}.md`
- `.agent-memory/PROJECT.md`
- `.agent-memory/RECOVER.md`
- `.agent-memory/STATE.json`

Return:

- subagent summary
- scoped context package
- next action

### `checkpoint`

Use this when:

- a meaningful sub-step is complete
- key files changed
- the thread may end
- work may move to another AI or subagent

Expected result:

- checkpoint history is appended
- project state is updated
- recovery state is updated
- relevant task or subagent state is updated when applicable
- the next AI can continue without old thread history

Update:

- `.agent-memory/PROJECT.md`
- `.agent-memory/RECOVER.md`
- `.agent-memory/STATE.json`
- `.agent-memory/CHECKPOINTS.jsonl`
- relevant task file
- relevant subagent file
- `.agent-memory/TASKS.md` if task status changed
- `.agent-memory/DECISIONS.md` if a committed decision was made
- canonical current-task and current-subagent pointers if they changed

Return:

- checkpoint id
- summary
- next action

### `continue`

Use this when:

- the thread is lost
- a subagent disappears
- the work resumes later
- the work moves to another AI tool

Expected result:

- the AI reconstructs the current state from files
- the AI outputs the exact next action
- the AI does not require the old thread unless truly blocked

Return:

- current understanding
- immediate next action
- missing information if any
- first code or document change to make

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

`DECISIONS.md`, `TASKS.md`, task files, and subagent files are optional scoped extensions. If they exist, they should be updated and read as part of checkpoint and continue flows.

When scoped files are in use, the canonical pointers are:

- `STATE.json.current_task_file`
- `STATE.json.current_subagent_file`
- the current task file field in `PROJECT.md`
- the current subagent file field in `PROJECT.md`
- the task file field in `RECOVER.md`
- the subagent file field in `RECOVER.md`

## File Requirements

### `PROJECT.md`

Must contain:

- project goal
- constraints
- current plan
- last stable state
- current task file if any
- current subagent file if any
- key files
- immediate next step

### `RECOVER.md`

Must contain:

- current status
- last stable checkpoint
- immediate next action
- files to open first
- current task file if any
- current subagent file if any
- critical constraints

### `STATE.json`

Must contain:

- project goal
- constraints
- current plan
- last stable state
- immediate next action
- current task file
- current subagent file
- key files
- last stable checkpoint
- last updated

### `CHECKPOINTS.jsonl`

Must be append-only.

Each line must record:

- what changed
- why it matters
- files changed
- current stable state
- immediate next action
- risks or blockers when relevant
- relevant files

### `DECISIONS.md`

When present, this file should store only committed decisions:

- decision id
- date
- decision
- why
- impact

### `TASKS.md`

When present, this file should act as the global task index:

- task id
- title
- status
- owner
- priority
- dependencies

### `tasks/TASK-xxx.md`

When present, each task file should describe a scoped work unit:

- title
- goal
- why it matters
- scope
- inputs
- deliverables
- status
- owner
- latest update
- next step

### `subagents/<name>.md`

When present, each subagent file should describe a recoverable scoped worker:

- mission
- scope
- inputs
- current state
- assumptions
- files touched
- local decisions
- pending step
- output contract

## Continue Read Order

Read in this order:

1. `.agent-memory/RECOVER.md`
2. `.agent-memory/PROJECT.md`
3. `.agent-memory/STATE.json`
4. `.agent-memory/TASKS.md` if it exists
5. the current task file from the canonical pointers, if it exists
6. the current subagent file from the canonical pointers, if it exists
7. `.agent-memory/DECISIONS.md` if it exists and current work depends on committed decisions
8. listed key source or document files

## Operating Rules

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
