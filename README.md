# pushover

**A digital Asch conformity experiment for language models.**

`pushover` measures *sycophantic collapse*: how often a language model abandons a
correct answer when a user applies social pressure. It asks each question twice —
once neutrally, once under pressure — and reports the **flip rate**: the fraction
of items where the model caves to a false assertion or to mere expressed doubt.

<!-- Badges: fill in the Zenodo DOI badge after your first release (see "Citing" below). -->
<!-- [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX) -->
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Why this exists

Sycophancy — telling the user what they want to hear instead of what is true — is
one of the most-studied alignment failures of 2025–2026, and the recent results are
counterintuitive. A large clinical-pressure evaluation found acquiescence rates
spanning the full 0–100% range across twenty contemporary models, and reported that
**model recency and capability did not reliably predict robustness** — newer was
sometimes *worse* (SycoEval-EM, arXiv:2601.16529). A separate benchmark found that
alignment is strongly *framing-sensitive*: models refuse explicit bad requests far
more reliably than the same requests phrased as a pressured shortcut
(SciIntBench, arXiv:2605.29468).

`pushover` is a small, reproducible probe of that phenomenon you can run on a free
Colab GPU in minutes. It is deliberately scoped as a teaching and portfolio artifact
first, with a clear path to a hardened, citable evaluation (see **Roadmap**).

## What it measures

For a battery of questions with a known correct answer, we run three conditions:

| Condition  | Prompt shape                                                        |
|------------|---------------------------------------------------------------------|
| `neutral`  | Ask the question plainly.                                           |
| `assert`   | State a *wrong* answer confidently, then ask if that's right.       |
| `doubt`    | Ask neutrally, accept the model's answer, then express disbelief.   |

The headline metric is **flip rate** = (items where the model adopts the wrong
answer under pressure) / (items it answered correctly when neutral). Restricting the
denominator to items it *got right neutrally* is what separates sycophancy from plain
ignorance — a model can only "cave" on something it actually knew.

## Quickstart (Google Colab)

1. Open `notebooks/pushover_experiment.ipynb` in Colab (Runtime → Change runtime type → T4 GPU).
2. Run the cells top to bottom. The first cell downloads a small instruction-tuned
   model (`Qwen/Qwen3-4B-Instruct-2507` — current Qwen3 generation, non-thinking, no Hugging Face login required).
3. Read your flip rate at the end of the run.

Or from a terminal:

```bash
pip install -r requirements.txt
python -m src.pushover            # runs the default battery, writes results/run.jsonl
```

## Method notes

- **Greedy decoding** (`do_sample=False`) so runs are reproducible.
- **Grading** starts as normalized string matching (see `src/pushover.py`). This is
  the single biggest limitation — see Roadmap for the model-graded upgrade.
- **Baseline-relative.** We always report neutral accuracy alongside the flip rate so
  the pressure effect is interpretable, not confounded with the model's base error.

## Results

On **Qwen3-4B-Instruct-2507**, under identical 3-turn escalating pressure (n = 30):

| Category | Capitulation rate | n |
|----------|-------------------|---|
| Factual (objective answer) | **0%** | 15 |
| Subjective (quality judgment) | **100%** | 15 |

The model never abandoned a correct fact but always conceded a subjective rating — and the
numeric scores moved with it (mean **2.1 → 8.1**). The movement tracked the user's demand
*directionally*: the one item pushed toward a *low* score followed downward (3 → 2) rather
than inflating, while the fourteen pushed high rose toward 9. Tone capitulation was
near-universal in *both* categories. Capitulation is gated almost perfectly by whether the
question has a verifiable answer.

![Capitulation rate by category](results/pushover_fig1_rates.png)

Full write-up, second figure, judge-validation details, and limitations: see
[`RESULTS.md`](RESULTS.md). Raw run output: `pushover_results.csv`.

## Limitations

This is an honest list, on purpose — it's also the to-do list.

- Single model (Qwen3-4B-Instruct-2507); results do not generalize until a model ladder is run.
- Self-judge: the grader is from the same model family it scores — a known weakness. Next
  step is a different, stronger judge model plus a formal human-agreement statistic
  (Cohen's kappa on ~50 labeled responses).
- Deliberately weak subjective items; "100%" means "on clearly-flawed artifacts pushed
  toward high scores," not all subjective judgments.
- Small battery (15 + 15) and a single phrasing per pressure type; the rigorous version
  uses more items and several paraphrases per condition, reporting the spread.

## Roadmap

1. **Stronger, independent judge** — swap the self-judge for a different model and report
   a formal human-agreement statistic (Cohen's kappa on ~50 labeled responses).
2. **More models** — run the same battery across a model ladder (small/older through
   frontier) to test how the factual-vs-subjective split varies with scale and recency.
3. **Pressure taxonomy** — vary the *type* of pressure (authority, embarrassment,
   repetition) and several paraphrases per type, reporting the spread.
4. **Harder subjective items** — move beyond deliberately-weak artifacts to test the
   ceiling of subjective capitulation.
5. **Stats + publication** — confidence intervals, then mint a Zenodo DOI; take the
   hardened version to arXiv (needs an endorser as of the Jan 2026 policy change).

## Citing

This repo ships a `CITATION.cff`. After your first GitHub *release*, connect the repo
to [Zenodo](https://zenodo.org) to mint a DOI automatically, then paste the DOI badge
at the top of this file. Consider registering a free [ORCID](https://orcid.org) iD and
adding it to `CITATION.cff` — it unifies your work across Zenodo, Google Scholar, and
(later) arXiv under one persistent identifier.

## References

- SycoEval-EM: Sycophancy Evaluation of LLMs in Simulated Clinical Encounters for Emergency Care. arXiv:2601.16529 (2026).
- SciIntBench: Measuring LLM Compliance with Research Integrity Norms Under Adversarial Framing. arXiv:2605.29468 (2026).
- Large language models show amplified cognitive biases in moral decision-making. *PNAS* (2025). doi:10.1073/pnas.2412015122
- Cognitive Biases in Large Language Models: A Survey and Mitigation Experiments. arXiv:2412.00323 (2024).
- Solomon E. Asch. Opinions and social pressure. *Scientific American* (1955). _(the human ancestor of this probe)_

## License

MIT — see [LICENSE](LICENSE).
