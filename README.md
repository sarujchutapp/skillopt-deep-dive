# SkillOpt, critically — do we need to *train* our skills?

A critical analysis of Microsoft Research's **SkillOpt** ([arXiv 2605.23904](https://arxiv.org/abs/2605.23904)),
which optimizes an LLM agent's skill document the way you would train a model —
gradient-like edits, a learning rate, and a validation gate — plus my own experiments
running it on an in-car voice-assistant benchmark.

> Research done during my Data Scientist internship at **Toyota Tsusho Mobility Informatics Pte. Ltd.**
> (Jul 2026). I read the paper against its own tables and prompts, ran it on the in-house
> benchmark from [ai-evaluation](https://github.com/sarujchutapp/ai-evaluation), and turned
> the result into a talk.

**Slides:** [PDF](docs/presentation/Presentation_SkillOpt.pdf) · [PowerPoint, with speaker notes](docs/presentation/Presentation_SkillOpt.pptx)

## The critique in brief

The paper is well engineered and its core idea is sound. The argument is that the evidence
for the *headline* claims does not survive checking against the paper's own data.

- **"Beats an expert human skill in 1–4 edits."** No run is ever seeded from the expert
  skill — seeds range from a 16-token stub to 516 tokens. The claim is "grew from a stub and
  finished above a separate document", not "improved an expert skill". One benchmark
  (LiveMath) supplies 28.5 of the 51.8 marginal points over the human skill; without it the
  advantage is +4.7.
- **Skills only grow.** Every learned skill ends larger than it started (×2.5 to ×53.6).
  The optimizer's prompts reward gap-filling and nothing penalizes length — yet skills exist
  to *save* context. Fix: a token-cost term in the acceptance gate.
- **Reusability and robustness are untested where it matters.** "Transfers across models"
  is GPT→GPT only; results come from one split seed with no variance reported.
- **Where it genuinely earns its keep:** weak or small target models. The hand-written skill
  goes net negative outside the GPT family, while SkillOpt lifts Qwen3.5-4B on ALFWorld from
  30.6 to 81.3. Rollout feedback finds failure modes no author would anticipate.

**Verdict — scoped, not dismissed.** For a small, stable domain with an expert available
and a strong target model, hand-write the skill. For a weak or small target model, train it.

Full argument with citations: [`docs/research/optimizer_necessity_critique.md`](docs/research/optimizer_necessity_critique.md).

## My experiments

Five SkillOpt runs on the in-car benchmark (Bedrock Sonnet as optimizer, local Ollama
models as the target; command track = tool calls, chat track = grounded answers).
Synthesis: [`docs/experiments/FINDINGS.md`](docs/experiments/FINDINGS.md) · per-run logs:
[`docs/experiments/`](docs/experiments/).

- **The seed prompt mattered more than the optimizer — at first.** One format fix to the
  starting prompt moved baseline accuracy 0.24 → 0.76; in runs 1–4 SkillOpt added nothing on
  top and slightly overfit.
- **Then a faithful run reversed it.** Run 5 matched the paper's setup (minimal seed,
  4 epochs, 2:3:5 split, scalar gate, a capable temperature-0 target): SkillOpt lifted
  held-out test accuracy 0.45 → 0.58 with no overfitting. The earlier verdicts were artifacts
  of my setup, not the method.
- **Aggregate scores hide mode collapse.** A flat "overall" score concealed the chat track
  falling 0.78 → 0.00 while commands improved; per-track reporting exposed it.
- **Small eval sets mislead.** At 9 items per track one item is 11%, so most early deltas
  were noise — growing the corpus came first.
- **Open question:** does it beat a strong *expert* seed? The four-cell experiment that would
  settle this is designed in the critique; no published paper reports it.

## Repository contents

| Path | What it is |
|---|---|
| [`docs/research/optimizer_necessity_critique.md`](docs/research/optimizer_necessity_critique.md) | The critique: thesis, assessment, mechanism notes, the decisive four-cell experiment |
| [`docs/research/slm_agent_research_directions.md`](docs/research/slm_agent_research_directions.md) | Frontier scan of small-model agents and text-space optimization |
| [`docs/research/dspy_overview.md`](docs/research/dspy_overview.md) | DSPy as a framework, and its optimizer menu |
| [`docs/experiments/`](docs/experiments/) | Five SkillOpt runs: seed, best skill, config, scores, cost, findings |
| [`docs/presentation/`](docs/presentation/) | The talk — PDF and PowerPoint (with speaker notes) |

The integration code (SkillOpt environments, data splits, Bedrock cost controls) lives in
[ai-evaluation/integrations/skillopt](https://github.com/sarujchutapp/ai-evaluation/tree/main/integrations/skillopt).
