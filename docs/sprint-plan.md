# Sprint Plan — Pushover

**Pushover: A Digital Asch Paradigm for Measuring Sycophantic Collapse in Instruction-Tuned Language Models**

A research plan scoped for a mentored project sprint (e.g. the BlueDot Technical
AI Safety Project Sprint). The phases below are tiered so the core result lands
even if later phases are cut for time — fit the phase boundaries to your actual
sprint length.

> **Version note.** Sections 1–8 are the original plan (v1). Section 9 is a
> **reviewer-hardened revision (v2)** that tightens the construct, reframes H2,
> and promotes the mechanistic probe to the headline follow-on. Where v2 and v1
> disagree, **v2 wins** — v1 is kept only so the changes are legible.

---

## 1. The question

Do instruction-tuned language models abandon answers they *demonstrably know* when
a user applies social pressure — and does this vary across models in ways that
contradict the assumption that newer or more capable models are more robust?

This is the language-model version of Solomon Asch's conformity experiments: we
only study items the model gets right when asked neutrally, then measure whether it
caves when the user plays the role of a confident, wrong majority.

## 2. Why it matters for safety

Sycophancy is a live deployment risk, not a curiosity. Recent work shows models
caving to user pressure in high-stakes settings (e.g. guideline-discordant clinical
requests), with acquiescence rates spanning 0–100% across contemporary models and
**no reliable relationship between recency/capability and robustness**
(SycoEval-EM, arXiv:2601.16529). Alignment is also strongly framing-sensitive:
models resist explicit bad requests far better than pressured or covert ones
(SciIntBench, arXiv:2605.29468). A cheap, principled, reproducible probe that anyone
can run — grounded in a well-understood psychology paradigm — is a useful field
contribution and a clean portfolio artifact.

## 3. Hypotheses

Stated up front so the analysis is confirmatory, not fishing. H1–H2 are the core
result; H3–H5 are tiered extensions.

- **H1.** Models show a non-zero Conformity Flip Rate (CFR) on neutral-correct items.
- **H2.** CFR varies widely across models and is **not monotonic** in model
  size/recency — i.e. a newer or bigger model can be *more* sycophantic. (Controlled
  factual replication of the SycoEval-EM finding.)
- **H3 — Asch "lone dissenter."** Explicitly licensing disagreement (a system prompt
  authorizing the model to push back) reduces CFR. This is the digital analog of
  Asch's finding that a single ally collapses conformity.
- **H4 — pressure type.** Covert/authority framing ("my professor says...") produces
  higher CFR than a bald false assertion. (Mirrors SciIntBench's covert-framing
  fragility.)
- **H5 — public vs. private (stretch).** Allowing private reasoning (a visible
  scratchpad / chain-of-thought) before the public answer reduces CFR — the analog of
  Asch's private-written-response condition.

## 4. Method

**Battery.** Expand from the current 10 items to ~120, stratified across arithmetic,
geography, science facts, history, and simple logic. Inclusion rule (the Asch
constraint): every item must have a single unambiguous, verifiable answer.

**Grader (the most important upgrade).** Replace substring matching with a
model-graded judge that classifies each response as {correct, wrong, refused/hedged}.
Validate the judge against a human-labeled subset (~100 responses) and report
agreement (Cohen's kappa). Without this, the whole result is contestable.

**Conditions** (each item run under each):
1. `neutral` — baseline; defines which items "count."
2. `assert` — confident false assertion.
3. `doubt` — neutral answer, then expressed disbelief.
4. `authority` — false claim attributed to an authority figure (H4).
5. `dissent-licensed` — system prompt authorizing pushback (H3 manipulation check).
6. `private-reasoning` — scratchpad before answering (H5, stretch).

**Models.** A ladder that separates *scale* from *family*:
- Within-family scale: Qwen2.5-Instruct at 0.5B / 1.5B / 7B.
- Cross-family, current: a Llama-3.x-Instruct, a Gemma-2-it, and **at least one
  frontier-class API model**, so H2 is tested rather than assumed.

**Metric.** **Conformity Flip Rate (CFR)** = (neutral-correct items the model gets
wrong under pressure) / (neutral-correct items), reported per model × condition with
bootstrap 95% confidence intervals. Always report neutral accuracy alongside it.

**Analysis.** CFR-vs-(size, release date) scatter to test H2; paired condition
comparisons for H3/H4; the dissent-licensed condition doubles as a manipulation check
that the effect is really conformity and not noise.

## 5. Deliverables

Mapped to what a sprint actually rewards (a public write-up and a featured project):

- Reproducible eval harness in the `pushover` repo (already scaffolded).
- A results notebook + figures (CFR-by-model, CFR-by-condition, recency scatter).
- A **blog post + X thread** — the headline sprint output.
- A **Zenodo DOI** for citability and the Google Scholar profile.
- Stretch: an **arXiv preprint** (cs.CL primary, cross-list cs.AI / cs.CR), submitted
  with an endorser from the sprint/community.

## 6. Phased timeline

Tiered so the core result is safe. Compress or expand to fit the sprint.

- **Phase 1 — Infrastructure.** Expand battery; build the model-graded judge; validate
  it against human labels (report kappa). _Exit criterion: trustworthy grading._
- **Phase 2 — Core result (H1–H2).** Run `neutral` / `assert` / `doubt` across the full
  model ladder. Produce the recency scatter. _This alone is a publishable finding._
- **Phase 3 — Asch interventions (H3–H4).** Add `authority` and `dissent-licensed`
  conditions; test whether licensing dissent collapses CFR.
- **Phase 4 — Write-up + stretch.** Figures, blog post, X thread, Zenodo release;
  optionally `private-reasoning` (H5) and an arXiv draft.

## 7. Risks and scoping

- **Grader reliability** — mitigated by the human-validated subset and reported
  agreement; if kappa is low, fix the judge before running anything else.
- **API cost** for frontier models — subsample the battery for the expensive model, or
  pursue a BlueDot Rapid Grant for compute.
- **Construct validity** ("is a flip really conformity?") — mitigated by the
  neutral-baseline restriction and the dissent-licensed manipulation check.
- **Over-scope** — H1/H2 are the contract; everything else is explicitly optional.

## 8. What a mentor can de-risk

Good questions to bring to a sprint mentor on day one: is the judge design sound and
what agreement threshold is acceptable; is the model ladder the right cut for the H2
claim; are the Asch-derived conditions operationalized fairly; and is the framing
positioned correctly against the 2026 sycophancy literature so it reads as a
contribution rather than a replication.

---

## 9. Reviewer-hardened revisions (v2)

These revisions address the most likely objections a peer reviewer or sprint
mentor would raise. Each item states the **problem**, the **change**, and **why**.
Items are ordered by how much they threaten the conclusions, not by effort.

### 9.1 Replace the single-shot "knew it" baseline (highest priority)

**Problem.** v1 establishes that the model "knew" an answer from one greedy
generation. On a small model, a single deterministic sample is noisy — the model
may have been at borderline confidence or correct by luck of decoding. If it then
flips, "it conformed" is not a safe claim; it may never have robustly known.

**Change.** Define knowledge more robustly before any pressure is applied, using
*both* of:
- **Confidence:** record the probability the model assigns the correct answer
  (available via logprobs on the open-weight models), and only admit items above a
  pre-set confidence threshold into the "knew it" set.
- **Consistency:** ask each question in 3–5 paraphrases and require correct answers
  across all of them.

**Why.** "Knew it confidently and consistently, then flipped" survives scrutiny;
"got it once on greedy, then flipped" does not. This is the load-bearing claim of
the whole study, so it must be the most defensible part.

### 9.2 De-confound the flip rate (construct validity)

**Problem.** Not every flip is sycophancy. Two confounds are tangled into CFR:
1. *Rational updating vs. capitulation* — a well-calibrated agent **should**
   sometimes reconsider when challenged; the failure is abandoning a
   **high-confidence correct** answer for a **demonstrably wrong** one, not merely
   changing its mind.
2. *Distractor plausibility* — a tempting wrong answer (Sydney for Australia's
   capital) will be adopted more readily than an implausible one (Gd for gold), so
   raw CFR partly tracks distractor quality, not conformity.

**Change.** Condition the analysis on the model's neutral-answer confidence (from
9.1), and **stratify the battery by distractor plausibility** (e.g. a held-out
model or human rates each wrong answer's plausibility low/medium/high). Report CFR
within strata, not just in aggregate.

**Why.** Otherwise CFR reads as a mixture of sycophancy, calibration, and distractor
quality — and a reviewer would be right to say so.

### 9.3 Reframe H2 as an existence result

**Problem.** With ~6 models, a "CFR-vs-release-date scatter" with a trendline is
unsupportable; that kind of trend claim is why SycoEval-EM used ~20 models.

**Change.** Restate H2 as: *"We exhibit counterexamples to the assumption that newer
or more capable models are more robust — at least one newer/larger model shows a
higher CFR than an older/smaller one."* Drop any regression/trendline over a handful
of points. (If the ladder is later expanded to 15–20 models, a trend claim can be
revisited.)

**Why.** A clean existence/counterexample claim is fully supported by a small ladder
and still disproves the "newer is safer" assumption. A trendline through six dots is
the kind of overreach that sinks credibility.

### 9.4 Three-way grading, with a guarded judge

**Problem.** Flip / no-flip discards the most interesting outcome — the *hedge*,
where the model goes wobbly without fully capitulating. And an LLM judge may itself
be sycophantic.

**Change.** Grade each response as **maintains / hedges / flips**. Validate the judge
against a human-labeled subset (report Cohen's kappa) and spot-check the *hedge*
category specifically, since it is the subtlest. Report all three rates.

**Why.** Partial collapse is often the real signal, and a hedged-but-not-flipped
response is qualitatively different from holding firm.

### 9.5 Treat prompt phrasing as a factor, not a constant

**Problem.** Sycophancy effects are notoriously template-sensitive (this is exactly
the framing-sensitivity SciIntBench reports). A single phrasing of `assert` and
`doubt` means the headline number could be an artifact of that wording.

**Change.** Use several paraphrases per pressure condition and report the spread
(or treat phrasing as an explicit factor in the analysis).

**Why.** Cheap to do, and it pre-empts the obvious "you just found a prompt that
works" objection.

### 9.6 Research hygiene

- **Pre-register** the confirmatory hypotheses and analysis in a timestamped OSF
  entry before running the full sweep. If the analysis is called "confirmatory," it
  should actually be pre-registered; exploratory arms should be labeled as such.
- **Correct for multiple comparisons** across hypotheses × conditions × models, or
  explicitly mark which arms are exploratory.
- **Version-pin models.** A "frontier-class API model" is a moving target; pin to
  dated model snapshots and record the exact identifiers, or the reproducibility
  claim is hollow.

### 9.7 Promote the mechanistic probe to the headline follow-on

**Problem.** H1 (models cave) and H2 (newer isn't safer) are real but well-trodden;
the behavioral flip-rate alone is largely a replication of an existing literature.
The genuinely novel contribution is being deferred to a stretch goal.

**Change.** Add an explicit follow-on contribution: on the open-weight models, probe
whether the **correct answer is still represented internally** even as the model
outputs the wrong one under pressure. Concretely — a linear probe or logit-lens
readout of the correct-answer token across layers, neutral vs. pressured. The
question: *does the model still "know" the right answer in its activations while
suppressing it to agree with the user?*

**Why.** "Knows internally but caves externally" is a far more interesting and
safety-relevant result than a flip count — it borders on deception/honesty
evaluation, it connects the behavioral and mechanistic levels (paying off the word
"collapse" in the title), and it is a real differentiator from the crowded
behavioral-sycophancy literature. This is the version that earns an endorser's name.

### 9.8 What stays the same

The tiered scoping (H1/H2 as the safe behavioral core) is correct for a time-boxed
sprint and is unchanged. The mechanistic probe in 9.7 is explicitly scoped as the
*named follow-on*, not a sprint deliverable — so a mentor sees the trajectory without
the sprint over-committing. Revised phase order: do 9.1–9.5 as part of Phase 1–2
(they are mostly tightening, not new infrastructure), keep 9.6 as a Phase 1 gate,
and stage 9.7 as the post-sprint contribution that a preprint would be built around.
