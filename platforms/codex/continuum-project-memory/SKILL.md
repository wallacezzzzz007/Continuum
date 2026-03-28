---
name: continuum-project-memory
description: Help AI agents create, update, and resume durable project memory under .agent-memory/ so work survives thread loss, resets, and handoffs with Continuum.
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
```

Task files and subagent files are optional. If they exist, update and read them. If they do not exist, do not invent extra complexity.

## Reference Templates

Use these bundled references as the preferred source when creating runtime files:

- [references/project-template.md](../../../references/project-template.md)
- [references/recover-template.md](../../../references/recover-template.md)
- [references/state-template.json](../../../references/state-template.json)
- [references/checkpoints-template.jsonl](../../../references/checkpoints-template.jsonl)

If a reference file is unavailable, create the runtime files directly using the field requirements in this skill.

## Init

Use this when:

- a new project starts
- an existing project adopts this system for the first time

### Required actions

1. Create `.agent-memory/` if it does not exist.
2. Create:
   - `.agent-memory/PROJECT.md`
   - `.agent-memory/RECOVER.md`
   - `.agent-memory/STATE.json`
   - `.agent-memory/CHECKPOINTS.jsonl`
3. Infer from the repository and user request:
   - project goal
   - key constraints
   - one immediate next step
4. Write the first stable checkpoint.

### Success condition

A fresh AI thread should be able to read `RECOVER.md` and immediately know what to do next.

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
5. If a current task file exists, update it.
6. If a current subagent file exists, update it.

### Every checkpoint must record

- what changed
- why it matters
- current stable state
- exact next step
- relevant files

### Quality bar

The next AI should not need the old thread to continue.

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
4. current task file if it exists
5. current subagent file if it exists

### Then produce

- current understanding
- exact next action
- files to inspect first
- any conflict between memory and current project state

### Continue rule

Do not ask for the full old thread unless truly blocked.

## Summary

This skill manages durable project state through files.

`init` creates the memory.

`checkpoint` preserves stable progress.

`continue` resumes from durable state.
