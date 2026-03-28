# Continuum

Continuum is a Codex skill for durable project state outside the chat thread.

It focuses on three actions:

- `init`
- `checkpoint`
- `continue`

The goal is simple: let work continue across thread loss, context reset, subagent failure, and switching between AI tools.

## Core Idea

The AI should not depend on thread history as the source of truth. Instead, it should create and maintain durable state under `.agent-memory/`.

## How It Works

`init`

- creates the memory files for a project
- writes the goal, constraints, and next step
- records the first stable checkpoint

`checkpoint`

- saves durable progress after meaningful work
- updates project state, recovery state, and checkpoint history
- keeps handoff and resume simple

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
2. generate the base files
3. write the project goal, constraints, and immediate next step
4. record the first checkpoint

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
4. current task file if one exists
5. current subagent file if one exists

Then the AI should output:

- current understanding
- immediate next action
- files to inspect first
- missing information only if necessary

## Why This Exists

- thread loss should not kill a task
- chat history should not be the only memory layer
- long-running work needs stable checkpoints
- different AI tools should be able to share the same project state

## Repository Structure

- `platforms/codex/continuum-project-memory/SKILL.md`
  Codex skill definition.
- `platforms/codex/continuum-project-memory/agents/openai.yaml`
  Codex UI metadata.
- `references/project-template.md`
  Project memory template.
- `references/recover-template.md`
  Recovery entrypoint template.
- `references/state-template.json`
  Machine-readable state template.
- `references/checkpoints-template.jsonl`
  Checkpoint log template.
- `docs/spec.md`
  Project-level specification and workflow notes.

## Status

The current version is intentionally small.

It defines the core behavior and the minimum file model needed to prove one thing:

an AI task can continue from durable project state instead of depending on chat memory.
