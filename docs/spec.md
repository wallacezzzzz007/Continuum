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
- the project goal, constraints, and next step are recorded
- the first stable checkpoint exists

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
- the next AI can continue without old thread history

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

## Runtime Files

```text
.agent-memory/
  PROJECT.md
  RECOVER.md
  STATE.json
  CHECKPOINTS.jsonl
```

Task files and subagent files are optional. If they exist, they should be updated and read as part of checkpoint and continue flows.

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
- current stable state
- exact next step
- relevant files

## Continue Read Order

Read in this order:

1. `.agent-memory/RECOVER.md`
2. `.agent-memory/PROJECT.md`
3. `.agent-memory/STATE.json`
4. current task file if it exists
5. current subagent file if it exists

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
