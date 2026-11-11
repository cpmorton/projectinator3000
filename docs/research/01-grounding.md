# 01. Grounding: platform facts and existing patterns

Checked 2026-11-11. Research notes; nothing here is a decision.
Facts are from pages fetched on 2026-11-11 unless marked [unverified].

## 1. Claude Code facts that constrain the design

| Topic | Fact | Source |
|---|---|---|
| AGENTS.md | "Claude Code reads `CLAUDE.md`, not `AGENTS.md`." Documented bridge: `@AGENTS.md` import inside CLAUDE.md (symlink also works, not on Windows). | code.claude.com/docs/en/memory |
| AGENTS.md standard | Stewarded by the Agentic AI Foundation (Linux Foundation). Read by Codex, Cursor, Copilot coding agent, Gemini CLI, Aider, goose, opencode, Zed, VS Code, etc. Claude Code is not on the list. Nearest file wins. | agents.md |
| Subagents | `.claude/agents/` (project), `~/.claude/agents/` (user), plugin `agents/`. Markdown + YAML frontmatter; `name`, `description` required; `tools`, `model`, `skills`, `isolation`, `effort`, etc. optional. | /en/sub-agents |
| Agent teams | Experimental, off by default (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`). Mailbox + shared task list under `~/.claude/`. Not formed in `-p` mode. One team per session, no nesting. | /en/agent-teams |
| Skills | `.claude/skills/<name>/SKILL.md` (project), `~/.claude/skills/` (user), plugin `skills/`. Slash commands merged into skills. Frontmatter includes `license`, `compatibility`, `context: fork`, `agent`, `hooks`, `paths`. | /en/skills |
| Hooks | 30+ events incl. `SessionStart`, `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStart/Stop`, `TaskCreated/Completed`, `PreCompact`. Handler types: `command`, `http`, `mcp_tool`, `prompt`, `agent` (experimental). Exit 2 blocks. | /en/hooks |
| Env vars | `env` block in settings.json; SessionStart hook can append `export X=y` to `$CLAUDE_ENV_FILE`. | /en/settings, /en/hooks |
| Settings precedence | managed > CLI > `.claude/settings.local.json` > `.claude/settings.json` > `~/.claude/settings.json`. Managed CLAUDE.md at `/etc/claude-code/CLAUDE.md`. | /en/settings, /en/memory |
| Plugins | `.claude-plugin/plugin.json`; a repo is a marketplace via `.claude-plugin/marketplace.json`; repo can pre-declare via `enabledPlugins` + `extraKnownMarketplaces` (after folder trust). | /en/plugins |
| Dev container | Official Feature `ghcr.io/anthropics/devcontainer-features/claude-code:1.0`. The reference `.devcontainer` in anthropics/claude-code is an example, not a maintained base image. | /en/devcontainer |
| Cloud sessions | Ubuntu 24.04 + per-environment setup script (root, ~5 min, must exit 0). "Replacing the base image entirely isn't supported yet." No devcontainer.json support mentioned. | /en/claude-code-on-the-web, /en/cloud-environments |
| Cloud carry-over | Carries: CLAUDE.md, `.claude/rules|skills|agents|commands`, repo settings.json hooks/permissions (single-repo sessions), `.mcp.json`. Does NOT carry: `~/.claude/*`, user plugins, repo `enabledPlugins`. | same |
| Headless | `claude -p`, `--output-format text|json|stream-json`, `--json-schema`, `--agents`, `--resume`. | /en/headless |
| Auth policy | OAuth is for "ordinary use of Claude Code and other native Anthropic applications". An end user may sign into the unmodified binary with their own plan. Third-party developers may not offer claude.ai login or route requests through plan credentials on behalf of users; SDK builders use API keys. `claude setup-token` / `CLAUDE_CODE_OAUTH_TOKEN` is documented for CI. Plan limits "assume ordinary, individual usage". | /en/legal-and-compliance, /en/authentication |

Implications:
- Only repo-scoped config (`CLAUDE.md`, `.claude/`, `.mcp.json`) reaches local, Codespaces AND cloud sessions. Homedir config and plugins do not reach cloud.
- "Same base image everywhere" cannot include Claude cloud today. The unit of reuse has to be an idempotent install script (or Dev Container Features) that the Dockerfile and the cloud setup script both call. The image is one packaging of it.
- A Go wrapper that shells out to the user's own logged-in `claude -p` sits on the permitted side as long as it never touches credentials. It must not be presented as "log in with Claude" in our product.

## 2. Dev container layering (containers.dev, GitHub docs, MS Learn)

- Features = installable units (`devcontainer-feature.json` + `install.sh`), published as OCI artifacts, layered onto any base image. Templates = files applied to a project.
- Prebuild: `devcontainer build --workspace-folder . --push true --image-name <img>:<tag>`; downstream uses `"image": "<img>"`. Dev container config and Feature metadata can be baked into image labels and are "automatically picked up when the image is referenced".
- Guidance: universal things in the prebuilt image, project-specific things in the downstream config.
- GitHub template repos copy the default branch's files with unrelated history. There is no upstream update path from a template.
- Codespaces prebuilds rebuild on push / schedule / config change; they run through `onCreateCommand` and `updateContentCommand` but not `postCreateCommand`.
- Azure Pipelines container jobs run every step inside a named image (`container:`), so one image can serve devcontainer and CI. GitHub Actions has the equivalent `container:` key [unverified today].

## 3. Orchestration patterns

- Anthropic "Building effective agents": prefer the simplest thing; workflows (predefined code paths) vs agents (self-directed). Named patterns: prompt chaining, routing, parallelization (sectioning, voting), orchestrator-workers, evaluator-optimizer.
- Anthropic multi-agent research system: lead agent + parallel subagents. Agents use ~4x chat tokens, multi-agent ~15x. Each subagent brief needs objective, output format, tool/source guidance, task boundaries. Poor fit when agents need shared context or tasks are tightly coupled; most coding has less parallelism than research.
- OpenAI Agents SDK: LLM-driven vs code-driven orchestration, mixable. "Agents as tools" (manager keeps control) vs "handoffs" (specialist takes over). Recommends specialised agents, evals, self-critique loops.
- Mapping to Mort's sketch: Supervisor = manager with agents-as-tools. Debate = evaluator-optimizer plus voting, with the orchestrator as moderator. Deterministic interview = prompt chaining with gates (a workflow, not an agent).
- Persona study (Harb et al. 2026): read in full; see [03-personas.md](03-personas.md). It tests single-agent prompting, not debate.

## 4. Patterns already present in existing repositories (candidates to extract)

Private repositories are described, not named.

| Pattern | Where |
|---|---|
| Public engine + private canon as two repos; `upstream` remote with push URL `DISABLED` | cursus and tiller-claude-toolkit (each with a private companion) |
| Toolchain as a versioned GHCR image; one `FROM` line pinned in `.devcontainer/Dockerfile`; Dependabot bumps it; CI reads the same line | private companion of cursus (documented in cursus `docs/PRIVATE-REPO.md`) |
| Idempotent `tools/setup.sh` called by both devcontainer `postCreateCommand` and a `SessionStart` hook | private companion of cursus (documented in cursus `docs/PRIVATE-REPO.md`) |
| `CLAUDE.md` = `@AGENTS.md` + repo specifics; procedures in `.agents/skills/` | daily-driver-os |
| Layered instructions: toolkit CLAUDE.md, private CLAUDE.md overrides | tiller-claude-toolkit and its private companion |
| ADRs with Status / Alternatives / "Revisit when" | daily-driver-os docs/decisions |
| Contract tests for invariants (bats); leak guards; pre-commit hooks | daily-driver-os, tiller, cursus |
| Hooks as guardrails and run ledger (PreToolUse guard, Stop hook, append-only ledger) | a private mail-triage repository |
| Routine = markdown spec + `prefs/` (learned) + `log/` (history); every lesson its own `Teach:` commit | a private daily-routine repository |
| Read-only by default; propose, approve, back up, write; "plans discussed in chat aren't facts" | tiller-claude-toolkit `CLAUDE.md` |
| Cloud setup script documented and versioned in repo; multi-repo sessions skip repo SessionStart hooks | tiller-claude-toolkit `docs/cloud-environment.md` |
| Machine-wide guardrails in `/etc/claude-code/managed-settings.json` | daily-driver-os ADR 0012 |
| `Assisted-by: <Model> via <Tool>` commit trailer | daily-driver-os AGENTS.md |

Note: `cpmorton/cpmorton` is only a profile README. The public-artifact release model found is cursus (public engine, GHCR image, private canon).
daily-driver-os was skimmed at README / file-listing depth only, as requested.

