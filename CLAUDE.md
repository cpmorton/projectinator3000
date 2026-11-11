@AGENTS.md

# Claude Code notes

Claude Code reads `CLAUDE.md`, not `AGENTS.md`; the import above is the documented
bridge (https://code.claude.com/docs/en/memory, checked 2026-11-11).

Anything in this file is, by definition, Claude-specific. Keep it short, and list every
Claude-only dependency in `docs/PLAN.md` under "Claude-only register" so the cost of
porting to another harness stays visible.
