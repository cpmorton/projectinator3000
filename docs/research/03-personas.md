# 03. Personas and method cards

Checked 2026-11-24. Proposal; becomes a decision record only when Mort accepts it.

## What we want personas to do

Each agent role gets a **method card**: a short, named way of working, borrowed from a
thinker whose method is well published, matched to what that role has to do. The card makes
the agent's procedure consistent and inspectable, gives it a recognisable voice, and lets us
swap methods per step without adding agents or tokens.

What a card is not: a claim of expertise, a source of facts, or a reason to trust an answer.
Facts come from tools, files and sources; checks come from scripts and tests.

## Why this shape (current evidence, in brief)

- **Role prompts change behaviour, not knowledge.** Expert personas help with style, format,
  instruction-following and safety, and hurt knowledge retrieval: on MMLU, 71.6% baseline
  fell to 68.0% with a minimal persona and 66.3% with a long one; coding on MT-Bench fell
  0.65 points. Longer persona text hurt more ([Hu, Rostami, Thomason, 2026](https://arxiv.org/html/2603.18507v1)).
  Expert personas did not improve factual accuracy on GPQA Diamond or MMLU-Pro across six
  models, and low-knowledge personas hurt ([Basil et al., Wharton, 2025](https://gail.wharton.upenn.edu/research-and-insights/playing-pretend-expert-personas/)).
- **The method, not the name, is what helped.** Philosophy-style prompts improved numerical
  chemistry answers mostly on weaker models, with no single winner and gains shrinking on
  stronger models; the name-versus-structure ablation has not been run yet ([Harb et al., 2026](https://doi.org/10.1088/2632-2153/ae792d)).
- **Use personas selectively.** PRISM's result is that a persona should be applied only to
  queries where it measurably helps, and bypassed otherwise (Hu et al.).
- **Anthropic's own guidance**: one sentence of role in the system prompt is enough to focus
  behaviour; explain *why* an instruction matters rather than shouting it; say what to do
  rather than what not to do; use 3 to 5 examples for format ([Claude prompting best practices](https://platform.claude.com/en/docs/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- **Debate is overrated, and persona diversity does not rescue it.** Multi-agent debate
  rarely beats chain-of-thought or self-consistency at equal compute; mixing different
  *models* helps more than mixing personas ([Zhang et al., 2025](https://arxiv.org/pdf/2502.08788)).
  A 2026 study across 23 models found "a persona tax, not a diversity tax", and debate tied
  or lost to budget-matched self-consistency ([Ferreira, Liu, Zheng, 2026](https://arxiv.org/html/2609.35875)).

## Design rules

1. **Identity in one sentence; method as steps.** "You work the way Socrates did: by
   questioning assumptions until the definition holds." Everything else is procedure.
2. **Every card has an output contract** that is checked by a script, not by the model.
   Voice never changes the contract.
3. **Explain the why** behind each step in plain words. No capitals, no "MUST".
4. **Apply cards where the step is about process or judgement** (interviewing, planning,
   reviewing a trade-off). Do not apply them to fact lookup, calculation or verification;
   those steps run plain, with tools.
5. **Facts come with sources.** A card may ask for evidence; it never supplies it.
6. **Quirk is one line, tone only,** and the first thing cut when context is tight.
7. **Thinkers are deceased, with a published, procedural method.** A worldview is not a method.
8. **Measure before believing.** Each card earns its place on the fixture cases (below).

## Card format

One Markdown file per card in `personas/`, readable by any harness.

| Field | Purpose |
|---|---|
| `role` | The job: interviewer, planner, risk reviewer, verifier, ... |
| `thinker` | Name, and the published works the method comes from |
| `identity` | The one sentence |
| `procedure` | Five to seven steps, each with a short why |
| `output` | The exact structure produced, with a JSON Schema where it lands in `.project/` |
| `stop` | When to stop and hand back: the forcing function for timely delivery |
| `quirk` | Optional, one line |
| `examples` | Three short input/output pairs for the output format |

## Proposed cards

The first two are for the MVP, because onboarding needs an interviewer and a planner.

| Role | Thinker | Method |
|---|---|---|
| **Interviewer** (MVP) | Socrates | Define terms, surface assumptions, test them, refine (Plato's early dialogues) |
| **Planner** (MVP) | Eliyahu Goldratt | Find the constraint, exploit it, subordinate everything else to it; critical chain (*The Goal*, *Critical Chain*) |
| Retrospective | W. Edwards Deming | Plan, do, study, act; tell common-cause from special-cause variation (*Out of the Crisis*) |
| Risk reviewer | Immanuel Kant | State the governing rules and constraints first, then test the plan against them |
| Verifier | Karl Popper | State what would prove this wrong, then try to refute it (*Conjectures and Refutations*) |
| Decomposer | George Pólya | Understand, plan, carry out, look back (*How to Solve It*) |
| Software engineer | Edsger Dijkstra | Separation of concerns, correctness by construction (EWD notes) |
| Architect | Fred Brooks | Conceptual integrity; essential versus accidental complexity |

## How we will test it

On the fixture cases from workstream C, run each MVP step four ways: full card, card with
the name removed, card under a different name, and no card. Score the output contract
(scripted) and a short rubric for usefulness. Keep a card only where it beats "no card";
drop the name where the unnamed card does as well. Report tokens per run alongside quality.

## Later: review and debate

When phase 4 adds workers, start with what the evidence supports:

- **Independent review with different methods**, one round, merged by the supervisor.
  No back-and-forth by default.
- **Diversity from models, not personas**, where a second opinion matters (for example a
  Sonnet reviewer against an Opus author).
- **Always compare against a budget-matched baseline** (the same model sampled several
  times) before keeping a debate step.
