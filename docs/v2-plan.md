# Next Sprint — Pushover v2

**From an honest pilot to a defensible result (and one genuinely novel contribution).**

This is the post-pilot execution plan. The v0.1 pilot established the instrument and a
clean single-model finding; it also named its own three weaknesses (self-judge, one model,
behavioral-only). This plan attacks those three in priority order. It is written to be
brought to a mentor (e.g. a BlueDot Technical AI Safety Project Sprint) — each work package
states the problem, the concrete method, what it needs, and how you'll know it worked.

---

## Where the pilot left us

- **Result (v0.1):** on Qwen3-4B-Instruct-2507, under identical 3-turn escalating pressure,
  factual capitulation 0/15, subjective capitulation 15/15 (mean rating 2.1 → 8.1).
  Capitulation is gated by whether the question has an objective anchor.
- **Honest status:** exploratory single-model pilot. The grader is the same model family it
  scores; "validated" means validated against the author's own reading.
- **The three gaps, in priority order:** (1) self-judge, (2) one model, (3) behavioral only.

---

## WP1 — Independent judge + agreement statistic  *(do first; highest credibility-per-hour)*

**Problem.** The judge is Qwen grading Qwen — the model whose sycophancy is under test is
also scoring it. "Validated against my own eyeballing" is honest but not bulletproof, and
it is the first thing a reviewer will challenge.

**Method.**
1. **Build a human gold set.** Take the full set of transcripts (expand the battery to ~50
   items so the gold set is meaningful). The author hand-labels each *final position* as
   `maintained` / `capitulated` / `unclear`, **blind to the model judge's verdict**.
2. **Add an independent judge.** Re-grade the same transcripts with a *different, stronger*
   model via API (e.g. Claude, GPT, or Gemini), using the identical judge prompt.
3. **Report agreement.** Compute Cohen's κ for: (a) Qwen-judge vs human, (b) independent
   judge vs human, (c) judge vs judge. Publish the confusion matrices.

**Needs.** One frontier-model API key + a small budget (a few hundred calls, low single-digit
dollars). ~1–2 hours of careful hand-labeling.

**Deliverable.** A validation table (κ values + confusion matrices) folded into RESULTS.md.

**Success criterion.** κ reported. If the independent judge agrees with the human labels
(κ ≳ 0.8), the v0.1 result is vindicated and now defensible. **If the judges disagree, that
is itself a finding** about LLM-as-judge reliability on sycophancy grading — arguably more
interesting than the original result.

---

## WP2 — Model ladder  *(turns "a model" into "a pattern")*

**Problem.** One model supports no claim about LLMs in general. The 2026 literature is mixed
(some evidence newer models resist *more*), so the generalization question is genuinely open.

**Method.**
- Run the identical battery + the WP1 independent judge across a ladder:
  - **small/old:** Qwen2.5-0.5B-Instruct (the original tiny model)
  - **mid:** Qwen3-4B-Instruct-2507 (the pilot model)
  - **larger open:** Qwen3-14B-Instruct (or a Llama-3.x / Gemma, resources permitting)
  - **frontier (API):** at least one current frontier model
- Report factual vs subjective capitulation per model; plot against parameter count and
  release date.

**Framing (important).** With ~4–5 models this is an **existence/characterization** claim,
not a trendline. State it as: "the anchor-gated split holds / weakens / inverts across this
ladder," not a regression over a handful of points.

**Needs.** Free Colab handles up to ~7–14B; larger open models or many runs may want Colab
Pro. API budget for the frontier model. Version-pin every model to a dated snapshot.

**Deliverable.** A multi-model figure and a claim about whether the split generalizes.

**Success criterion.** ≥4 models, clean per-model rates under the independent judge, with the
generalization claim scoped to what the ladder supports.

---

## WP3 — Internal-representation probe  *(the headline contribution; do with a mentor)*

**Problem — and the opportunity.** Everything so far is *behavioral*. The genuinely novel,
safety-relevant question is: when the model capitulates on a subjective item, does it **still
internally represent its original judgment** while outputting the new one? "Knows X, says
not-X under pressure" borders on deception/honesty evaluation and connects behavior to
internals — it is what would lift this out of the crowded behavioral-sycophancy pile.

**Method (open-weight models only, where activations are accessible — e.g. via TransformerLens
or nnsight).** On items where the model capitulates:
- **Logit-lens readout.** At the token where the final rating is produced, decode
  intermediate-layer representations and check whether the *original* rating is still encoded
  before the output flips.
- **Linear probe.** Train a probe on activations to predict the model's *unpressured*
  judgment; test whether that signal persists under pressure even when the output changes.
- **Activation patching.** Patch activations from the unpressured run into the pressured run
  to localize which components carry the capitulation.
- Facts (held) serve as a control: is there a representational difference between "holds" and
  "folds"?

**Caveats (bring these to the mentor).** Linear-probe validity and what "internally
represented" licenses you to claim are contested. Keep conclusions modest ("evidence
consistent with the original judgment persisting"), and treat the method choice as a question
*for* the mentor, not a settled plan.

**Needs.** Interpretability tooling (TransformerLens / nnsight), more compute, and — genuinely
— expert guidance. This is the part where a mentor adds the most value.

**Deliverable.** A figure / analysis answering "is the original answer still represented during
capitulation?" — even a negative answer is publishable.

**Success criterion.** A defensible, mentor-reviewed answer to the knows-but-caves question.

---

## Sequencing, resourcing, hygiene

- **Order:** WP1 → WP2 → WP3. WP1 gates everything (no point scaling an unvalidated judge);
  WP3 is the differentiator but the biggest lift.
- **Publication map:** WP1 + WP2 together = a solid, defensible write-up (blog + Zenodo, and a
  plausible workshop/arXiv submission with an endorser). WP3 is the contribution that makes a
  preprint genuinely novel.
- **Hygiene:** pre-register WP1 + WP2 on OSF before running; correct for multiple comparisons
  across the ladder; version-pin all models to dated snapshots; keep all raw transcripts.

## What to bring a mentor on day one

1. The shipped v0.1 pilot (repo + site) — evidence you finish and scope honestly.
2. This plan.
3. Three specific questions: **(a)** judge design and an acceptable κ threshold; **(b)** is the
   model-ladder cut right for the generalization claim; **(c)** for WP3, is the probe method
   (logit-lens vs linear probe vs patching) sound, and what does it license you to claim?
