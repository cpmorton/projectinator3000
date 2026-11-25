# AGENTS.md

Working notes for any agent in this repository. Harness-agnostic: nothing in this
file may depend on one vendor's tooling. Claude-only notes go in `CLAUDE.md`.

## Read first

1. `docs/CHARTER.md`: scope, MVP, Definition of Done. If a request falls outside it, say so.
2. `docs/PLAN.md`: current phase, workstreams, open decisions.

## Gates

- **Chat is not evidence.** A plan discussed in conversation is not built until it is in
  the repository. Verify against files before claiming something exists.
- **Verify against current official docs** before writing a command, path, config key or
  flag. Cite the URL and the date checked in `docs/research/` or the decision record.
- **Definition of Done.** Every piece of work states its Definition of Done before it
  starts and is checked against it before it is called finished.
- **Decisions belong to Mort.** Recommend with trade-offs; record a decision in
  `docs/decisions/` only after Mort makes it. Do not pre-write decision records.
- **Public repository.** No secrets, tokens, personal data or private project details.
- **Idempotent scripts.** Anything under `scripts/` must be safe to run twice.
- **Bash first, Go where warranted.** No PowerShell except where Windows automation
  requires it. No Python or Node for project tooling.

## Layout

See "Proposed repository layout" in `docs/PLAN.md`. It is a proposal until merged work
makes it real; do not create empty directories to match it.
