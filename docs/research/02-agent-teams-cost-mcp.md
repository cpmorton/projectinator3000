# 02. Agent teams, cost forecast, and what MCP is for

Checked 2026-11-11. Research notes; nothing here is a decision.

## 1. Subagents, agent teams, and what the difference means for us

Source: https://code.claude.com/docs/en/agent-teams, https://code.claude.com/docs/en/sub-agents

| | Subagents | Agent teams |
|---|---|---|
| Shape | Hub and spoke. The main agent spawns a worker, the worker returns a result | Mesh. A lead plus teammates that are each a full Claude Code session |
| Communication | Result goes back to the caller | Teammates message each other directly through a mailbox, and share a task list |
| Coordination | Main agent manages everything | Self-coordination; teammates claim unblocked tasks |
| Cost | Lower: only a summary returns to the main context | Higher: "approximately 7x more tokens than standard sessions when teammates run in plan mode" |
| Status | Stable | Experimental, off by default (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) |

What is useful to know before designing around teams:

- A teammate loads the project's `CLAUDE.md`, MCP servers and skills, plus the lead's spawn
  prompt. It does not inherit the lead's conversation. So the repository is the shared
  context, which is the same discipline this project needs anyway.
- A teammate can be spawned from a subagent definition. A role defined once in
  `.claude/agents/` works as a subagent today and as a teammate later. That makes
  "design for teams, ship with subagents" cheap.
- Each teammate can run a different model. The docs recommend Sonnet for teammates.
- Quality gates are hooks: `TeammateIdle`, `TaskCreated`, `TaskCompleted`. Exit code 2
  blocks and sends feedback. A Definition of Done check belongs on `TaskCompleted`.
- The documented best fit is exactly the debate pattern: "Debugging with competing
  hypotheses", with teammates told to "try to disprove each other's theories, like a
  scientific debate". Also parallel review with distinct lenses.
- Recommended size is 3 to 5 teammates with 5 to 6 tasks each.
- Limits that matter here: teams do not form in `-p` (headless) mode or the Agent SDK; no
  session resume for in-process teammates; one team per session; no nested teams; the
  lead is fixed; split panes are not supported in the VS Code integrated terminal; task
  status can lag. Team state lives under `~/.claude/`, not in the repository, and there is
  no project-level team config file.

Reading for this project:

- The supervisor/worker sketch in the charter is the **subagent** shape. It needs nothing
  experimental.
- The debate gate is the **agent team** shape. It is interactive-only and experimental, so
  it cannot be a dependency of anything deterministic or headless. Treat it as an optional
  mode a user turns on, never as plumbing.
- Because team state is not in the repository, our own state (`.project/`) stays the
  record. A team is a way to do work, not a place to keep it.

## 2. Cost forecast

Prices per million tokens, from the Claude API pricing table as cached on 2026-11-03.
Re-check https://platform.claude.com/docs/en/about-claude/pricing before relying on them.

| Model | Input | Output | Cache read | Cache write (5 min) |
|---|---|---|---|---|
| Fable 5.2 | $8.00 | $40.00 | $0.20 | 1.25x input |
| Opus 5.5 | $4.00 | $20.00 | $0.20 | 1.25x input |
| Sonnet 5.5 | $2.00 | $10.00 | $0.10 | 1.25x input |
| Haiku 5.5 | $0.10 | $0.50 | about 0.1x input | 1.25x input |

Batch processing is 50% off everything, for work nobody is waiting on.

### Reference points from the documentation

- Claude Code on API pricing averages about $13 per developer per active day, $150 to
  $250 per month, and stays under $30 per active day for 90% of users (enterprise
  deployments; https://code.claude.com/docs/en/costs).
- An agent uses about 4x the tokens of a chat, and a multi-agent system about 15x
  (https://www.anthropic.com/engineering/multi-agent-research-system). So multi-agent is
  roughly 4x a single agent.
- Agent teams are about 7x a standard session with teammates in plan mode.

### A model of our own workloads

These are estimates from stated assumptions, not measurements. Phase 1 should replace them
with real numbers from `/usage` and from `claude -p --output-format json`.

**Solo working session**: one hour, 40 requests, about 60k tokens of context re-read from
cache each time, 3k new input and 1.5k output per request, two cold cache rebuilds.

| Model | Cache reads | New input | Rebuilds | Output | Total |
|---|---|---|---|---|---|
| Fable 5.2 | $0.48 | $1.20 | $1.20 | $2.40 | about $5.30 |
| Opus 5.5 | $0.48 | $0.60 | $0.60 | $1.20 | about $2.90 |
| Sonnet 5.5 | $0.24 | $0.30 | $0.30 | $0.60 | about $1.45 |
| Haiku 5.5 | $0.02 | $0.02 | $0.02 | $0.03 | about $0.08 |

**One debate gate**: three workers, two rounds, each worker given a 25k-token brief and
writing 2k per round, moderated by an Opus 5.5 supervisor.

| Workers on | Workers | Moderator | Total per gate |
|---|---|---|---|
| Opus 5.5 | $0.72 | $0.18 | about $0.90 |
| Sonnet 5.5 | $0.36 | $0.18 | about $0.55 |
| Haiku 5.5 | $0.02 | $0.18 | about $0.20 |

**A full agent team for an hour** at 7x a solo Opus 5.5 session: about $20. With Sonnet
teammates, roughly half that.

### What the numbers say

- For daily interactive use, a flat subscription beats the API. Twenty solo Opus 5.5
  sessions a month is about $58 on the API. Check current plan prices at
  https://claude.com/pricing.
- On a subscription the scarce resource is the usage window, not dollars. Subagents and
  teammates draw from the same allowance, so a 7x team burns a five-hour window fast.
- On the API, worker fan-out is cheap if the workers are small models. A debate gate with
  Haiku 5.5 workers costs cents, and the moderator dominates.
- So the affordable architecture is a hybrid, later: the subscription for the interactive
  supervisor, and an API key with a hard workspace spend limit for headless worker
  fan-out. Not before phase 4.
- The debate should be a gate at a few decision points per project (charter sign-off, plan
  sign-off, a contested trade-off), not the default loop.

### A trap to design out

In `-p` mode, "the key is always used when present": an `ANTHROPIC_API_KEY` in the
environment silently moves headless runs from the subscription to per-token billing
(https://code.claude.com/docs/en/authentication). `scripts/doctor.sh` should report which
credential a run will use, and the budget ledger should record it.

## 3. What MCP can do for us

Source: https://code.claude.com/docs/en/mcp, https://code.claude.com/docs/en/costs

MCP is the harness-agnostic **tool** layer in the same way `AGENTS.md` is the
harness-agnostic **instruction** layer. Most agent harnesses speak it. That is the real
argument for it here, more than any single feature.

Where it would earn its place:

1. **Typed changes to project state.** Tools such as create ticket, update ticket, record
   spend and check Definition of Done validate against the JSON Schemas and write
   atomically, so no agent hand-edits JSON. This is the strongest case.
2. **Structured input.** MCP elicitation lets a server ask the user for structured
   answers, which suits a deterministic interview. Caveat from the docs: it "is served
   only in interactive terminal sessions", so it cannot be the only path.
3. **Cross-project reporting.** One server reading the state of several repositories.
4. **Portability of the above** to any harness that speaks MCP.

Constraints that shape the design:

- Project scope is `.mcp.json` at the repository root. It travels with the repository, and
  interactive sessions ask once before using it.
- A local stdio server is a child process: no daemon, no port. A single static Go binary
  needs no Node or Python runtime. That is the beginner-stability argument, and the reason
  Go is warranted for this piece.
- Tool definitions are deferred by default, so an idle server costs little context. Output
  over 10,000 tokens warns and 25,000 is the default cap, so tools must return small,
  structured results.
- The costs page is blunt that CLI tools "are still more context-efficient than MCP
  servers". So build the CLI first.

Proposed shape, for phase 4: one Go binary, `pj`. The CLI is the product; hooks, skills
and CI call it. `pj mcp` is a thin adapter exposing the same operations over stdio. Until
then, bash and `jq` behind the same command names, so nothing that calls them has to change.

What MCP should not be used for here: instructions, procedures or personas. Those stay as
files.
