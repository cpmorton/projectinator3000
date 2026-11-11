# Charter

Draft 1, 2026-11-11. Owner and decider: Mort (`cpmorton`). A charter, not a constitution:
one page, revised when reality disagrees with it.

## Problem

Most people, including people in IT, use an AI model as a chat box and start from zero
every time the work is bigger than a chat. They have no repeatable way to turn "I want to
do X" into a planned, tracked, finished project with an AI doing real work alongside them.

## What this is

An open-source golden path: a repository you start from, plus the teaching that goes with
it, that gets a curious person into a working VS Code + Claude Code + GitHub setup and
walks them from an idea to a planned project with a Definition of Done.

It is three things, in this order of importance:

1. **A process**: a short, repeatable interview that produces a charter, a Definition of
   Done and a first plan, then a rhythm of plan / do / check / look back. Lean and agile
   practice without the vocabulary.
2. **A repository shape**: where instructions, skills, state and scripts live so that any
   agent, in any supported environment, picks up the project cold.
3. **An environment**: idempotent setup that later becomes a container base layer shared
   by dev container, CI and other runtimes.

## Who it is for

Persona zero is a T-shaped generalist with strong product and project management
instincts and enough dev/DevOps skill to be comfortable in a terminal. In practice: Mort.

The eventual audience runs from curious lay people to engineers. The MVP does not serve
them yet; it must not make choices that lock them out.

## Principles

- **Extract, don't invent.** Mort's existing repositories already contain most of the
  patterns. Harvest them critically, update to current practice, generalise.
- **Harness-agnostic by default.** Instructions in `AGENTS.md` and portable skill files,
  state in plain JSON, tooling in bash and Go. Every Claude-only dependency is registered.
- **Deterministic where it can be.** Scripts, schemas and hooks do what does not need
  judgement. The model does what does.
- **Repo-scoped beats user-scoped.** A user's personal preferences must not change how the
  project's tooling behaves. Where a machine setting is unavoidable, guide the user through
  it and check it.
- **Definition of Done, always.** Stated at the start of every unit of work and checked at
  the end, from the first interview through the retrospective.
- **Affordable.** The MVP runs on one Claude Pro subscription with a single agent.
  Multi-agent work is designed for, measured, and switched on only where it earns its cost.
- **Teach while doing.** Every step that asks something of the user explains why.

## MVP

A person creates a repository from this one, opens it in a supported environment, and runs
onboarding. Onboarding interviews them, writes a charter with a Definition of Done and a
first plan as structured state in the repository, and leaves a clean project shell with
instructions for what to do next.

Supported environments for the MVP, in order: Claude Code on the web (prompts and context
only), GitHub Codespaces, Linux, and WSL configured to behave like Linux.

One agent. No worker team, no debate, no model abstraction, no container image.

### Definition of Done for the MVP

1. **It onboards itself.** Running onboarding on this repository produces this project's
   own charter and plan state, and that output is what the project then runs on.
2. **It passes the fixture cases.** A small set of simple, well-defined sample projects
   (at least one non-coding) each go from a one-line goal to a charter, a Definition of
   Done and a plan without hand-editing files.
3. **A fresh environment works.** From an empty Codespace, setup is one command, is
   idempotent, and a check script reports green.
4. **Checks run in GitHub Actions**: script lint, schema validation of the state files,
   and the fixture comparisons.
5. **The Claude-only register is complete** and each entry says what the portable
   equivalent would be.

## Not in the MVP

Worker agents and the supervisor/worker runtime; persona debate; agent teams; an MCP
server; any model-provider abstraction; the dev container image and its CI; AKS or other
compute; cross-project reporting; Docker Desktop on Windows, ever.

These are sequenced in [PLAN.md](PLAN.md), not abandoned.

## How we will know it is working

- Mort uses it for the next real project instead of starting from a blank chat.
- A second person gets through onboarding without live help.
- Time from "I have an idea" to "there is a plan with a Definition of Done in a
  repository" is under an hour.

## Constraints

- Public repository, Apache-2.0. No secrets or personal data.
- Personal use on Mort's own subscription. The project never ships a login flow or routes
  anyone's requests through plan credentials.
- Bash first; Go where warranted (CLI, automation, APIs); GitHub Actions for testing.
- Starts as Mort's portfolio piece. It may later become a professional tool; nothing in the
  MVP should depend on that.
