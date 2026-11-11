# Plan

Draft 1, 2026-11-11. Companion to [CHARTER.md](CHARTER.md). Everything here is a proposal
until work merged to `main` makes it real.

## Phases

Each phase has one Definition of Done. A phase is not started until the one before it is
done, but workstreams inside a phase can run in parallel.

| Phase | Outcome | Definition of Done |
|---|---|---|
| **0. Charter** (this pull request) | Charter, plan and research agreed | Mort has edited and merged this pull request |
| **1. Walking skeleton** | Conventions, state schema, onboarding interview v0, setup and check scripts, lint CI. Prompts and files only | This repository onboards itself, and the fixture cases pass in GitHub Actions |
| **2. Handoff** | Onboarding turns a fresh copy of this repository into a clean project shell; on-ramp guides for Claude Code on the web, Codespaces, Linux and WSL | A fixture case goes from "Use this template" to a working shell in a new Codespace with no hand-edited files |
| **3. Base layer** | The idempotent setup becomes a published container image; downstream pins it in one `FROM` line; CI runs in the same image; machine guardrails baked in | One commit passes the same checks in Codespaces, local WSL/Linux and a GitHub Actions container job |
| **4. Team** | Supervisor plus two workers with method cards, a budget ledger, an optional debate gate, the `pj` Go CLI and its MCP adapter | A fixture case is planned and executed by the team inside a stated token budget, and the result is checked against its Definition of Done |
| **Later** | Model-provider abstraction, agent teams, cross-project reporting, AKS/compute targets, a tutorial track for lay users | Sequenced when phase 4 is done |

MVP = phases 1 and 2.

## Workstreams for phase 1

One workstream = one project thread = one pull request = one Definition of Done.

| ID | Workstream | Depends on | Definition of Done |
|---|---|---|---|
| A | **Skeleton and conventions**: directory layout, `AGENTS.md`, the skills location and symlink, `.vscode/` recommendations, lint CI (shellcheck, markdown, JSON Schema) | Phase 0 | CI is green on an otherwise empty skeleton; the Claude-only register exists |
| B | **State model**: JSON Schemas for charter, plan and ticket; how Definition of Done is represented and checked; generated read-only views | A | Schemas validate in CI; this project's own charter is expressed in them |
| C | **Onboarding interview**: the question script, its gates, the skill that runs it, the fixture cases and their expected outputs | B | Fixture cases pass; self-onboarding reproduces the charter from B |
| D | **On-ramps spike** (research, then docs): Claude Code on the web setup script, Codespaces, automatable WSL without Docker Desktop, symlink behaviour | Phase 0 | A verified, dated guide per environment and one idempotent `scripts/setup.sh` plus `scripts/doctor.sh` |
| E | **Method cards**: the persona/method card format, first cards, and the evaluation design from [research/03-personas.md](research/03-personas.md) | Phase 0 | Two cards used by the onboarding skill; evaluation plan written, not run |
| F | **Critical harvest**: a pattern catalogue from Mort's repositories with a keep / revise / drop verdict and the current-practice reason for each | Phase 0 | Catalogue merged; each "keep" names the workstream that will adopt it |

A goes first. B then C are coupled and can share a thread. D, E and F can start any time
after phase 0 and run in parallel with everything.

### Working across project threads

- A thread starts cold. The repository and project memory are the only shared context, so
  every thread reads `AGENTS.md`, the charter and this plan first, and ends by updating
  the status of its row here.
- Do not run two threads that edit the same files. The table above is cut so that A/B/C
  touch tooling and D/E/F touch `docs/`.
- A thread that discovers a decision for Mort stops and asks; it does not decide and
  document.
- Research threads (D, E, F) produce a dated document with sources. Build threads produce
  a pull request with passing checks.

## Proposed repository layout

```text
AGENTS.md                    # canonical, harness-agnostic rules
CLAUDE.md                    # "@AGENTS.md" plus Claude-only notes
README.md
docs/
  CHARTER.md  PLAN.md
  decisions/                 # one record per decision actually made, with "Revisit when"
  research/                  # dated, sourced grounding
  onramps/                   # per-environment setup guides (workstream D)
.agents/skills/<name>/SKILL.md   # canonical procedures: onboard, charter, dod-check, retro
.claude/
  settings.json              # hooks and env, committed
  skills -> ../.agents/skills    # symlink, as daily-driver-os already does
  agents/                    # subagent definitions (phase 4, Claude-only)
.project/                    # this project's own machine-readable state
  charter.json  plan.json
  tickets/T-0001.json        # one file per ticket
  schema/*.schema.json
personas/                    # method cards (workstream E)
template/                    # the payload that becomes an onboarded project's shell
scripts/                     # idempotent bash: setup.sh, doctor.sh, hook entry points
tests/                       # bats for bash; fixtures/cases/<name>/ with expected output
cmd/pj/  internal/           # Go, phase 4: one binary, CLI first, `pj mcp` as an adapter
.vscode/                     # extensions.json, settings.json, tasks.json
.github/workflows/           # lint and test now; image build in phase 3
.devcontainer/               # phase 3
.mcp.json                    # phase 4
```

Notes on the choices:

- **State as JSON with a schema**, per Mort. Recommended refinement: one file per ticket
  plus a small `plan.json`, not a single file. Parallel threads and agents then touch
  different files, diffs stay readable, and each file maps to a database row later.
  Human-readable views are generated and never hand-edited.
- **`template/` inside this repository** for the MVP. Publishing it as a separate template
  repository built by CI is the "public artifact" model from cursus and belongs in phase 3.
- **Hooks call scripts; skills call scripts.** Anything deterministic lives in `scripts/`
  (later `pj`), so the same check runs from a hook, a skill, a terminal and CI.

## Where configuration lives

| Layer | Holds | Why |
|---|---|---|
| Repository | `AGENTS.md`, `CLAUDE.md`, `.claude/`, `.agents/skills/`, `.mcp.json`, `.project/` | The only layer that reaches local, Codespaces and Claude Code on the web alike |
| Image (phase 3) | Tools, `/etc/claude-code/managed-settings.json`, managed `CLAUDE.md` | Managed settings have the highest precedence, so user preferences cannot override project guardrails |
| User home in the container | Credentials only | The container's home is built, not inherited; the host's `~/.claude` is not mounted |
| User's own machine | As little as possible; `scripts/doctor.sh` checks what is unavoidable | Determinism |

Until phase 3 there is no image, so phase 1 and 2 rely on the repository layer and on
`doctor.sh` telling the user what on their machine differs.

## Claude-only register

Every dependency on Claude Code specifically, and its portable equivalent.

| Dependency | Portable equivalent |
|---|---|
| `CLAUDE.md` with `@AGENTS.md` | `AGENTS.md` is read natively by most other agents |
| `.claude/settings.json` hooks | The scripts they call; other harnesses wire them their own way, CI runs them regardless |
| `.claude/skills` discovery | `.agents/skills/` holds the files; the format is plain Markdown with front matter |
| `.claude/agents/` subagents (phase 4) | Method cards in `personas/` are plain Markdown any agent can be given |
| Claude Code on the web as an on-ramp | Codespaces |

## Open decisions for Mort

Kept short on purpose. Each has a recommendation; none blocks merging this draft.

1. **Template location.** This repository is the template for the MVP (recommended), or a
   separate generated template repository from day one.
2. **State layout.** One file per ticket plus `plan.json` (recommended), or a single file.
3. **Persona picks.** Accept the first two method cards proposed in
   [research/03-personas.md](research/03-personas.md), or choose others.
4. **When to add an API key.** Not before phase 4, and then only for headless worker
   fan-out with a hard spend limit (recommended). See
   [research/02-agent-teams-cost-mcp.md](research/02-agent-teams-cost-mcp.md).
