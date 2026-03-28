# Continuum

Continuum is a Codex skill for durable project state outside the chat thread.

It focuses on five actions:

- `init`
- `create_task`
- `spawn_subagent`
- `checkpoint`
- `continue`

The goal is simple: let work continue across thread loss, context reset, subagent failure, and switching between AI tools.

## Core Idea

The AI should not depend on thread history as the source of truth. Instead, it should create and maintain durable state under `.agent-memory/`.

## Templates

The canonical templates live in the repository root under `references/`.

The copies under `platforms/*/continuum-project-memory/references/` exist only to keep the skill self-contained.

## How It Works

`init`

- creates the memory files for a project
- writes the goal, constraints, and next step
- records the first stable checkpoint

`create_task`

- creates a scoped task file and updates the task index
- establishes the canonical current task pointer

`spawn_subagent`

- creates a scoped subagent file when work is split
- establishes the canonical current subagent pointer

`checkpoint`

- saves durable progress after meaningful work
- updates project state, recovery state, and checkpoint history
- keeps task and subagent state in sync when they exist

`continue`

- reads project memory in a fixed order
- reconstructs the current state
- gives the next action without relying on old thread history

## What `init` Means

Use `init` when:

- a new project starts
- an old project uses this system for the first time

The AI should:

1. create `.agent-memory/`
2. generate the base files, including optional task and decision scaffolding
3. write the project goal, constraints, and immediate next step
4. record the first checkpoint

## What `create_task` Means

Use `create_task` when:

- work should be tracked as a scoped unit
- the project needs a canonical current task file

The AI should:

1. create `.agent-memory/tasks/{task-id}.md`
2. update `TASKS.md`
3. update the current task pointers in `PROJECT.md`, `RECOVER.md`, and `STATE.json`

## What `checkpoint` Means

Use `checkpoint` when:

- a sub-step is complete
- key files changed
- the thread may end
- work is being handed to another AI or subagent

The AI should:

1. append `CHECKPOINTS.jsonl`
2. update `PROJECT.md`
3. update `RECOVER.md`
4. update `STATE.json`
5. update the current task file or subagent file if one exists
6. update `TASKS.md` or `DECISIONS.md` when task state or committed decisions changed

## What `continue` Means

Use `continue` when:

- the thread is lost
- a subagent disappears
- the work moves to another AI
- work resumes the next day

The AI should read in this order:

1. `RECOVER.md`
2. `PROJECT.md`
3. `STATE.json`
4. `TASKS.md` if it exists
5. the current task file if one exists
6. the current subagent file if one exists
7. `DECISIONS.md` if current work depends on committed decisions
8. listed key source or document files

Then the AI should output:

- current understanding
- immediate next action
- first code or document change to make
- files to inspect first
- missing information only if necessary

## What `spawn_subagent` Means

Use `spawn_subagent` when:

- a narrow part of the work should be delegated

The AI should:

1. create `.agent-memory/subagents/{name}.md`
2. include only the scoped context needed by that subagent
3. update the current subagent pointers in `PROJECT.md`, `RECOVER.md`, and `STATE.json`

## Why This Exists

- thread loss should not kill a task
- chat history should not be the only memory layer
- long-running work needs stable checkpoints
- different AI tools should be able to share the same project state

## Status

The current version is intentionally small, but it now supports optional scoped task and subagent state on top of the core recovery files.

It defines the core behavior and the minimum file model needed to prove one thing:

an AI task can continue from durable project state instead of depending on chat memory.
