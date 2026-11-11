# 03. Personas and method cards

Checked 2026-11-11. Research notes; nothing here is a decision.

## 1. What the study found

Harb et al., "The ballad of LLM agents: philosophical reasoning for chemistry",
*Mach. Learn.: Sci. Technol.* 7 (2026) 030503, https://doi.org/10.1088/2632-2153/ae792d.
Prompts and data: https://github.com/HassanHarb92/Sci_reasoning_LLMs.

- **Setup.** Seven "philosophy agents" (Socrates, Aristotle, Descartes, Hegel, Hume,
  Plato, Kant), each a fixed system prompt encoding a reasoning method. Tested on 243
  open-ended numerical chemistry questions from ChemBench, on GPT-4o, GPT-5 and GPT-5.1.
  Single turn, no tools, no memory.
- **Headline.** At a strict 1% error threshold: +21.8 points for GPT-5.1 with Socrates,
  +11.5 for GPT-4o with Hume, +4.5 for GPT-5 with Kant.
- **Gains shrink as the base model gets stronger.** GPT-5 started at 86.4% and most
  agents moved it a few points. Hegel and Plato made it slightly worse.
- **No universal winner.** The best agent depends on the model and the task. Descartes
  (decomposition) led on calculation, Kant (premises and constraints) on rule-bound
  reasoning, Hume (evidence) on pattern and example questions, Socrates (questioning
  assumptions) on rescuing weaker models.
- **Three rules of thumb** from the authors: align the method with the model's
  capability; match the method to the dominant cognitive demand; avoid one universal
  prompting strategy.

What the study does **not** show, in the authors' own words:

- It is not a multi-agent or debate study. Their selection diagram is "a conceptual
  roadmap rather than a validated multi-agent workflow". Routing and deliberation are
  future work.
- The philosopher labels are "shorthand for specific prompt behaviors". They have not yet
  run the ablation that separates the name from the structure (same principles under a
  different name, philosophical vocabulary with the structure broken). So there is no
  evidence yet that the name itself does anything.
- Small subgroups: "a single question can shift the reported accuracy by several
  percentage points".
- One benchmark, one model family, numerical chemistry. Nothing about planning or
  project management.

## 2. What that means for this project

- **The method is the asset; the name is a mnemonic.** What transfers is an explicit
  reasoning procedure with an output contract. A named thinker is a compact, memorable
  handle for it, and a fair way to add voice.
- **Expect modest gains on a frontier model.** We run on strong models, where the paper
  found the smallest and least reliable effect. The honest claim is "a consistent,
  inspectable procedure", not "better answers".
- **Match the card to the job's cognitive demand**, which is Mort's instinct too: choose
  the thinker by what the agent has to do.
- **For the MVP this is free.** One agent switching method cards at different steps is the
  single-agent setting the paper actually tested. No extra agents, no extra tokens beyond
  the card.
- **We can run the missing ablation ourselves**, cheaply, on our fixture cases: full card,
  card with the name removed, card with a different name, no card. It settles whether the
  persona earns its tokens, and it is a publishable result for a portfolio.

## 3. The method card

One plain Markdown file per card, usable by any harness. Proposed fields:

| Field | Purpose |
|---|---|
| Role | The job this card is for (interviewer, planner, risk reviewer, verifier) |
| Thinker | Name, and the published works the method comes from |
| Demand | The cognitive demand it matches (assumption testing, decomposition, rule checking, evidence weighing, refutation) |
| Procedure | Five to seven numbered steps, written as actions |
| Output contract | The exact structure the step must produce. Never affected by voice |
| Stop rule | When to stop and hand back. This is the forcing function for timely delivery |
| Quirk | One line of voice. Optional, and the only place chaos is allowed |

Rules for choosing a thinker:

- Deceased, with a published, well-documented method. No living people.
- The method must be procedural enough to write as steps. A worldview is not a method.
- The quirk costs a sentence, lives in tone only, and is dropped first if tokens are tight.

## 4. Candidates by role

Recommendations, for Mort to accept or replace. The first two are proposed for the MVP,
because onboarding needs exactly an interviewer and a planner.

| Role | Thinker | Method, and where it is published | Why it fits |
|---|---|---|---|
| **Interviewer** (MVP) | Socrates | Elenchus: define terms, surface assumptions, test them, refine (Plato's early dialogues) | The paper's router and critic; an onboarding interview is assumption testing |
| **Planner** (MVP) | Eliyahu Goldratt | Theory of Constraints, five focusing steps; critical chain (*The Goal*, *Critical Chain*) | Finds the long pole and sequences around it, which is the planning job in the charter |
| Decomposer | George Pólya | Understand, plan, carry out, look back (*How to Solve It*) | The clearest published problem-solving procedure there is; ends with a retrospective |
| Risk reviewer | Immanuel Kant | State the governing rules and constraints first, then test the plan against them | The paper's rule-based agent; maps to security, legal and policy review |
| Verifier / tester | Karl Popper | State what would prove this wrong before doing it, then try to refute it (*Conjectures and Refutations*) | Domain-relative test-first for any field, not only code |
| Evidence checker | David Hume | Ground each claim in observation, grade confidence, distrust unsupported causation | The paper's empirical agent; this is the fact-check role |
| Software engineer | Edsger Dijkstra | Separation of concerns, correctness by construction, simplicity (the EWD notes) | A computer-science thinker with a method and no shortage of quirk |
| Architect | Fred Brooks | Conceptual integrity, essential versus accidental complexity (*The Mythical Man-Month*, "No Silver Bullet") | Guards scope and coherence |
| Retrospective | W. Edwards Deming | Plan, do, study, act; separate common-cause from special-cause variation (*Out of the Crisis*) | The look-back ritual, with a method for not overreacting to one bad run |

Alternatives worth a look: Taiichi Ohno (five whys, go and see) for root cause, and
Christopher Alexander (pattern languages) for the template catalogue itself.

## 5. Debate, later

When phase 4 adds workers, the moderator pattern from the charter maps onto documented
mechanics rather than anything bespoke:

- Workers with different cards review the same artefact independently, then see each
  other's positions once. Two rounds, fixed.
- The supervisor moderates: asks each for the strongest objection to the others, applies
  the stop rule, and either decides or names the one open question for the user.
- Every claim that survives goes through the evidence checker before it reaches the user.
- Cost is in [02-agent-teams-cost-mcp.md](02-agent-teams-cost-mcp.md): cents per gate with
  small-model workers, so the constraint is the usage window, not money.
