# Universal FDE

Universal FDE guides an AI agent and a human through training a real business workflow, one verified stage at a time. It does not assume that the final system must be fully automated.

## Two authoritative documents

```text
Human-facing guide + Agent interaction reference
        ↓
Human ↔ Agent collaboration / interaction protocol

FDE_TRAINING_SKILL.md
        ↓
Authoritative Stage / Gate / State / execution protocol
```

- [Human-facing guide + Agent interaction reference](<FDE_Human-facing guide + Agent interaction reference.md>) explains the human–agent interaction protocol and provides the human entry prompt.
- [FDE_TRAINING_SKILL.md](FDE_TRAINING_SKILL.md) defines the authoritative sequential training protocol, including Stages, Gates, project state, evidence, and execution rules.
- The Codex skill entrypoint is in `.codex/skills/universal-fde/SKILL.md`. When this repository is used as a Codex workspace, `AGENTS.md` points relevant workflow-training tasks to it.
- [LICENSE](LICENSE) permits non-commercial use; all commercial use requires prior written authorization.

## Start

Open this repository in Codex and ask to train a new real business workflow, or use the Start Prompt in the human-facing guide. The agent will read the interaction protocol, initialize project state, retain the Global Constitution, and work through only the current Stage and its required artifacts. Later Stages are loaded only after the current Gate passes.


