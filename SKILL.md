---
name: universal-fde
description: Train or improve a real business workflow with Universal FDE. Use when the user wants an AI agent to understand, document, validate, prototype, or operationalize a real ToB or enterprise process using the project's staged Stage and Gate protocol.
---

# Universal FDE Progressive Training

Use this skill only for a workflow-training project. The two project documents have distinct authority:

- `FDE_Human-facing guide + Agent interaction reference.md` defines the human–agent collaboration and interaction protocol. Read it for the interaction approach and human-facing prompts.
- `FDE_TRAINING_SKILL.md` is authoritative for Stages, Gates, `FDE_PROJECT_STATE`, evidence, and execution. Never let the guide override it on runtime protocol.

## When Universal FDE starts a new workflow

1. Read the Human-facing guide + Agent interaction reference for the human–agent interaction protocol.
2. Initialize `FDE_PROJECT_STATE` in the user's active project/workspace, not in this reusable skill repository.
3. Read and retain `GLOBAL CONSTITUTION` from `FDE_TRAINING_SKILL.md`.
4. Retrieve only `FDE_STAGE_00_INIT` from `FDE_TRAINING_SKILL.md`.
5. Begin the Stage 00 interview using the minimum critical questions needed to understand the real business. Introduce the process briefly; ask grounded, answerable questions, not a long questionnaire.
6. Do not design the final automation architecture yet.
7. Do not preload later stages. After the current Gate passes, update `FDE_PROJECT_STATE` and retrieve only the next Stage and its declared `REQUIRED_ARTIFACTS`.

## Resuming an existing workflow

Read the existing `FDE_PROJECT_STATE`, retain the `GLOBAL CONSTITUTION`, and retrieve only the recorded current Stage plus its declared `REQUIRED_ARTIFACTS`. Do not reset a project to Stage 00 unless the protocol or evidence requires backtracking. After a Gate passes, update state before retrieving only the next Stage.

## Selective retrieval is mandatory

`FDE_TRAINING_SKILL.md` contains future Stages that must not be preloaded. Use search or bounded file reads to locate and retrieve the `GLOBAL CONSTITUTION`, the current Stage section, and only artifacts that section declares as required. Do not read the whole file as a shortcut. Preserve unknowns as unknowns, separate facts from assumptions, and obey Gate and backtracking rules in the authoritative skill document.
