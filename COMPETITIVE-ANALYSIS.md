# Competitive & Improvement Analysis — `ml-oncology-benchmarks`

> Scope: documented, open, leakage-aware ML benchmarks for oncology (survival prediction, molecular
> subtyping) on open/aggregate/de-identified cancer data. Medium risk. Cancer guardrails: open data
> only, per-source licence verify, provenance, rigorous no-leakage methodology, honest baselines.
> Analysis date: 2026-06-29. Plan reviewed: PLAN.md v0.1.0.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong: it leads with the right thesis (leakage, not weak models, is the
cardinal sin), makes patient-grouped splitting and a mandatory external cohort defaults, encodes the
leakage audit as blocking CI, and treats the **optimism gap** (internal-CV minus external-cohort) as
the signature output. That framing is well-supported by the literature — Kapoor & Narayanan
catalogue eight leakage types across 294 papers in 17 fields ([Patterns 2023](https://www.sciencedirect.com/science/article/pii/S2666389923001599)),
and a 2026 study shows supervised feature screening and normalization on the *full* dataset inflates
even elastic-net baselines in cancer drug-response prediction ([bioRxiv 2026](https://www.biorxiv.org/content/10.64898/2026.02.05.704016v1.full)).
Still, there are concrete gaps, errors, and weak spots:

**A. Survival-metric correctness — partially specified, with real traps left open.**
- The plan names Harrell + Uno C-index, time-dependent AUC, and integrated Brier score (IBS) — good.
  But it does not state the deeper, now-well-documented problem: **the C-index is not a proper
  scoring rule for t-year predicted risks** and is implicitly time-dependent, so model *selection*
  by C-index can pick a worse model ([Hartman et al., *Stat Med* 2023](https://onlinelibrary.wiley.com/doi/full/10.1002/sim.9717);
  [practical-perspective review, *J Biomed Inform* 2020](https://www.sciencedirect.com/science/article/pii/S1532046420301246)).
  The spec should make **IBS / time-dependent calibration the primary headline** and C-index a
  secondary discrimination summary, not the reverse.
- **Uno's C-index requires an estimate of the censoring distribution and a truncation time tau**; the
  plan does not mandate declaring tau, the IPCW weighting scheme, or how tau is chosen on the
  external cohort (where the censoring distribution differs). Undeclared tau is itself a silent
  researcher-degree-of-freedom. Add it to `metricSpec`.
- **Calibration is mentioned only for classification.** For survival, calibration (e.g. D-calibration,
  calibration-in-the-large at fixed horizons) is at least as important as discrimination and is the
  half that C-index ignores. Missing.
- **No confidence intervals / uncertainty quantification.** The plan reports point estimates and a
  "reproduction tolerance," but never mandates bootstrap CIs on the metrics or on the optimism gap.
  Two models within overlapping CIs should not be ranked; a leaderboard without CIs invites exactly
  the cherry-picking the project exists to stop.

**B. Cross-validation rigor — under-specified.**
- M2 mentions "nested CV" for survival but M1 (subtyping) does not. **Nested CV (or a held-out outer
  loop) must be the default for *both*** because hyperparameter selection on the same folds used for
  reporting is itself a leakage channel. Make nested/outer-held-out CV a methodology-spec requirement,
  not a per-milestone afterthought.
- **No multiple-comparison / multiple-submission control.** A leaderboard is a multiple-testing engine:
  N teams probing one external cohort will produce an over-fit "winner" by chance. The plan has no
  mention of a **held-back, never-released final test cohort**, submission-count limits, or
  randomization-test / FDR control on leaderboard rankings. This is a notable omission for a project
  whose whole pitch is statistical honesty.
- **Single external cohort = single point of failure.** The plan mandates "an" external cohort. One
  external cohort can itself be idiosyncratic (its own batch signature). Where feasible, require
  **≥2 external cohorts** or leave-one-cohort-out (LOCO) evaluation so the optimism gap is not an
  artifact of one unlucky validation set.

**C. Batch/site confounding — the check is named but the hardest part is hand-waved.**
- The audit lists a "batch/centre confounding check" that tests whether the split or label is
  confounded with a detectable batch variable and "requires it be addressed or declared." But the
  cardinal failure mode is that **batch is confounded with the *biological label itself*** (e.g.
  TCGA tissue-source-site correlates with subtype), which no split policy can fix and which ComBat-style
  harmonization can *worsen* by leaking. The deep-learning histology literature shows models routinely
  learn the *site*, not the biology ([Howard et al., *Nat Commun* 2021](https://www.nature.com/articles/s41467-021-24698-1);
  ["batch effect not prognostication," *Cancer Cell* 2022](https://www.cell.com/cancer-cell/fulltext/S1535-6108(22)00522-0)),
  and TCGA itself carries strong centre batch effects ([*PMC* 2019](https://pmc.ncbi.nlm.nih.gov/articles/PMC6686424/)).
  The plan needs an explicit **"can the batch variable predict the label as well as the features do?"
  negative-control / permutation baseline**, and a rule for when harmonization is performed inside vs.
  outside the CV fold (ComBat fit on full data is a classic leak).

**D. Overfitting to TCGA — acknowledged as a risk but not structurally prevented.**
- TCGA is the candidate pool's centre of gravity (open tier via GDC/cBioPortal). The risk table notes
  "overfitting to TCGA" only obliquely. Because **TCGA-derived subtype *labels are themselves often
  defined on TCGA*** (circularity: training and validating a signature on the same cohort is the
  MAQC/SEQC cautionary tale), the methodology spec should forbid using a TCGA-defined label as ground
  truth *and* TCGA expression as features unless the external cohort uses an **independently derived
  label**. Otherwise the "external" GEO cohort may inherit TCGA's label definition and the
  independence is illusory. The plan flags cohort independence as a domain-reviewer duty but gives no
  concrete label-provenance test.

**E. Dataset versioning / drift — present but thin.**
- Good: SHA-256 + Wayback snapshot, retrieval timestamp, "harness detects data drift." Missing: GDC
  data is **re-released (data-release versions, e.g. harmonized vs. legacy; GENCODE re-annotations)**;
  cBioPortal studies are re-curated. The provenance triple should pin **the GDC data-release version
  and the cBioPortal study version**, not just a URL+hash, because the same accession changes content
  across releases. Also no policy for **what happens to published numbers when a source is corrected
  or withdrawn** (re-run? annotate? retract the leaderboard row?).

**F. Honest baselines — direction is right, specifics missing.**
- "Baselines exist to make the benchmark honest" is the correct stance. But the plan never enumerates
  the **non-trivial honest baselines that catch leakage**: (i) a **clinical-only / covariate-only
  baseline** (age, stage) — if the fancy genomic model can't beat age+stage, the benchmark says so;
  (ii) a **label-shuffle / permutation baseline** that should score ~chance (a positive control for
  leakage — if it scores well, you have a leak); (iii) a **batch-only baseline** (predict label from
  centre/platform alone). These three should be *mandatory* baselines, not optional.

**G. Subtyping label correctness.**
- "Established subtype label" is under-defined. Many subtype systems (PAM50, CMS, etc.) are
  **classifier-derived, not gold-standard**, and are platform/normalization-sensitive — re-deriving
  them on a new platform is itself a modelling step with its own leakage surface. The plan should
  require the label's **definition algorithm, version, and platform assumptions** be recorded in
  `labelSpec`, and flag when the label is itself a model output.

**H. Smaller gaps.**
- Reproduction tolerance is listed as an open question but **shipping M1 requires a number** — the
  Definition-of-Shipped depends on "within tolerance" yet tolerance is undefined; this is a
  circular dependency that should be resolved in M0.
- No mention of **class-imbalance handling leakage** (SMOTE/resampling applied before the split is a
  classic leak) in the preprocessing-isolation check — add resampling to the list of fit-on-train-only
  transforms.
- The **funded-lane budget cap** is noted but no benchmark is sized; some omics CV runs are
  non-trivial. Fine for a donated cold-start, but size at least the M1 pilot.
- "verifiedNeed:false until adopter secured" is honest and correct, but the **entire success metric
  set is gated on an unsecured adopter** — M0 outreach is the true critical path and deserves to be
  the highest-priority task, which it currently is not framed as.

Overall: methodologically the strongest of the cancer plans reviewed; the gaps are about
*statistical depth* (proper scoring, calibration, CIs, multiple-comparison control, mandatory
positive/negative-control baselines) rather than missing structure.

---

## 2. Competitive landscape

No existing project occupies the exact niche (open + leakage-audited-in-CI + mandatory-external-cohort
+ provenance-triple oncology benchmark). But several adjacent efforts overlap and set the bar.

**Crowdsourced cancer challenges (the closest "honest external validation" precedent)**
- **Sage Bionetworks / DREAM Challenges** — the gold-standard model for *blinded* external validation.
  The **Breast Cancer Prognosis Challenge (BCC)** had 354 teams predict overall survival on METABRIC
  (1,981 samples) with a blinded validation set; the **mCRPC prostate** and **NSCLC immunotherapy**
  challenges followed.
  *Strengths:* genuinely blinded holdout, real-time leaderboard, source-code transparency, community
  ensemble robustness ([*Sci Transl Med* 2013 / PMC](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3897241/);
  [mCRPC, *Lancet Oncol* 2017](https://pubmed.ncbi.nlm.nih.gov/27864015/)).
  *Weaknesses / gaps:* challenges are **episodic** (run once, then static), often on **controlled or
  consortium-gated data**, are **not packaged as a re-runnable leakage-audited harness**, and the
  leakage controls are implicit in the organizers' holdout rather than codified as portable CI tests.
  Elyos can be the *standing, open, re-runnable* complement to DREAM's episodic events.

**Survival ML libraries & benchmarks (the metric/baseline layer)**
- **scikit-survival** — the canonical sklearn-compatible time-to-event library (Cox, RSF, gradient
  boosting; Harrell/Uno C-index, time-dependent AUC, IBS) ([docs](https://scikit-survival.readthedocs.io/en/stable/user_guide/evaluating-survival-models.html)).
  *Strength:* correct, well-documented metrics — likely the project's metric backbone. *Gap:* a
  library, **not a benchmark** — no datasets, no splits, no external-cohort policy, no leakage audit.
- **pycox / DeepSurv / DeepHit, TorchSurv** — deep survival models and the KKBox benchmark ([TorchSurv](https://opensource.nibr.com/torchsurv/)).
  *Gap:* model zoo + one non-cancer benchmark; no oncology leakage governance.
- **soda-inria/survival-analysis-benchmark** — an exploratory comparative survival benchmark repo
  ([GitHub](https://github.com/soda-inria/survival-analysis-benchmark)). *Strength:* honest comparative
  intent. *Gap:* exploratory, not oncology-specific, no licence/de-id gate or external-cohort mandate.
- **SurvivalEVAL** — open-source survival-evaluation package (concordance, calibration, Brier)
  ([AAAI-SS](https://ojs.aaai.org/index.php/AAAI-SS/article/download/27713/27486/31764)). Useful
  upstream dependency for the calibration gap noted in §1.

**General ML benchmark infrastructure (the harness/leaderboard pattern to emulate)**
- **OpenML / OpenML Benchmarking Suites (e.g. OpenML-CC18)** — machine-readable tasks with fixed
  train/test splits, shared results, >1,500 studies powered ([arXiv 1708.03731](https://arxiv.org/abs/1708.03731);
  [*Patterns* 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12416095/)).
  *Strength:* the reproducible-task + curated-suite model is exactly the "definition-object-first"
  pattern the plan adopts. *Gap:* **domain-agnostic, no survival/censoring support, no de-id/licence
  gate, no leakage audit, no batch-confound notion.** Elyos = "OpenML suite, but oncology-aware and
  leakage-audited."

**Open cancer data platforms (the data substrate, not benchmarks)**
- **NCI Genomic Data Commons (GDC) open tier** and **cBioPortal** — the open expression+clinical
  substrate the plan targets ([cBioPortal, *Cancer Discov* 2012 / PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC3956037/)).
  *Strength:* large, harmonized, open-tier, widely cited. *Gap (and the project's opportunity):* they
  **distribute data, not benchmarks** — no fixed splits, no external-cohort pairing, no honest-baseline
  reference. They are upstream, not competitors.

**The reproducibility/leakage critique literature (the project's intellectual base — and validators)**
- **Kapoor & Narayanan, leakage taxonomy** ([*Patterns* 2023](https://www.sciencedirect.com/science/article/pii/S2666389923001599))
  — the foundational framework; the audit suite should map each check to a taxonomy entry.
- **Cancer drug-response leakage** ([bioRxiv 2026](https://www.biorxiv.org/content/10.64898/2026.02.05.704016v1.full))
  — empirical proof that whole-dataset normalization/screening inflates results.
- **Radiomics reproducibility critiques** — overfitting, batch/ComBat dependence, single-institution
  non-generalization; feature selectors that excel in training collapse on external sets
  ([*Diagn Interv Radiol* 2024](https://www.dirjournal.org/articles/reproducibility-and-interpretability-in-radiomics-a-critical-assessment/doi/dir.2024.242719);
  [*arXiv* reproducibility in medical-imaging ML, 2022](https://arxiv.org/pdf/2209.05097)).
- **Histopathology patient-overlap leakage** — predictive performance inflated by **up to ~41%** when
  tiles from the same patient land in both train and validation; patient-wise splitting essential
  ([Hatfield/melanoma exploration](https://sjhatfield.github.io/Kaggle-melanoma-contest-exploration/);
  [PANDA-PLUS-Bench, *arXiv* 2025](https://arxiv.org/pdf/2512.14922)). Direct empirical support for the
  plan's patient-grouped-split default.
- **Kaggle cancer competitions** (Histopathologic Cancer Detection/PCam, melanoma) — show both the
  pattern (public leaderboards drive engagement) and the failure mode (leaderboard overfitting, dataset
  duplicate leakage). *Gap vs. Elyos:* Kaggle has no de-id/licence governance, no external-cohort
  mandate, no provenance triple; the leaderboard *is* the multiple-testing engine §1.B warns about.

---

## 3. Gaps we can fill

1. **A standing, open, re-runnable leakage-audited oncology benchmark** — DREAM's blinded rigor but
   continuous and portable, not a one-off event, and on fully open data (no consortium gate).
2. **Leakage controls as committed, portable CI tests** — nobody else ships patient-overlap,
   preprocessing-isolation, batch-confound, duplicate-sample, and temporal-leakage checks as
   golden-fixtured CI that *fails the build*. This is genuinely novel infrastructure.
3. **The optimism gap as a first-class published output** — surfacing internal-CV→external drop for
   100% of benchmarks. No existing oncology benchmark reports this systematically; it is the honest
   number the literature keeps hiding.
4. **A de-identification + licence + access-tier gate as code** — OpenML/cBioPortal/GDC don't gate on
   redistribution-of-derived-metrics or NC-inheritance; Elyos turns SPDX + access-tier into a blocking
   artifact.
5. **Provenance triple (code, data-release, split-hash) on every number** — closes the "AUC 0.99 with
   no reproducible lineage" gap that the drug-response and radiomics critiques expose.
6. **Survival done *properly* for a benchmark** — calibration + IBS + IPCW-tau + CIs + positive/negative
   control baselines, packaged, which scikit-survival (library) and Kaggle (no censoring) don't provide.
7. **Mandatory honest control baselines** (clinical-only, label-shuffle, batch-only) as a reusable
   pattern — a teaching artifact the field lacks.

---

## 4. Differentiators to win

- **"Leakage fails CI" is the moat.** A benchmark that *mechanically refuses to publish a leaky number*
  is categorically different from a challenge that hopes organizers caught the leak. Lead every
  communication with this.
- **Honesty-as-the-product (the optimism gap).** Reframes "disappointing external numbers" from
  embarrassment to deliverable. This is a defensible position no leaderboard-win culture (Kaggle,
  SOTA-chasing) can copy without contradicting itself.
- **Fully open + redistributable-by-construction.** By excluding controlled data and gating on
  derived-redistribution rights, the artifacts are freely reusable in teaching/review — a niche DREAM
  (gated data) and dbGaP-based studies cannot occupy.
- **Reviewer/editor utility.** Position as the **reference harness a journal reviewer re-runs** to
  check a prognostic-model claim — a constituency (reviewers, reproducibility groups) no competitor
  serves directly.
- **TRIPOD-AI-aligned reporting + datasheets + model cards** baked in — speaks the language clinical
  prognostic-model reviewers already expect, lowering adoption friction.
- **Negative/positive controls shipped** (label-shuffle, batch-only) make the benchmark *self-auditing*,
  which doubles as the strongest teaching example for newcomers.

---

## 5. Claude API leverage

**Where Claude clearly helps (code, docs, audits-as-code, reporting):**
1. **Write the leakage-audit suite and its golden fixtures.** Claude is strong at generating the
   patient-overlap, preprocessing-isolation (static-analysis + runtime `fit`-assertion), duplicate-
   fingerprint, and temporal-leakage checks plus *deliberately-leaky* and *clean* fixture pairs that
   the CI must fail/pass — exactly the kind of well-specified, test-driven code where the spec is the
   methodology document.
2. **Implement baselines and the metric layer.** Cox/RSF/gradient-boosting and clinical-only /
   label-shuffle / batch-only control baselines wired to scikit-survival/lifelines; IPCW C-index with
   declared tau, time-dependent AUC, IBS, survival calibration, bootstrap CIs. Claude writes the
   plumbing; **the numbers come from running the code, never from Claude.**
3. **Generate provenance/reporting artifacts.** Datasheets-for-Datasets and Model Cards from the
   definition object, TRIPOD-AI-aligned report scaffolds, the methodology spec prose, and a
   provenance-ledger generator — high-volume structured writing Claude excels at.
4. **Licence/access-tier triage *drafting*.** Claude can pre-fill the gate checklist (locate the
   licence text, propose SPDX id, draft the access-tier classification and attribution string) to
   accelerate the human reviewer — as a *draft*, never the decision.
5. **Results-reporting & explanation.** Turn computed result tables into honest prose (including
   articulating *why* an optimism gap is large), and draft the researcher-facing docs.

**Where Claude must NOT decide (hard guardrails, mirroring the plan):**
- **No benchmark number ever originates from the LLM.** Every metric is computed by committed code
  against gated data and carries the (code, data-release, split-hash) triple. Claude-suggested numbers
  are forbidden; a fabricated or "estimated" result is a critical violation.
- **Leakage clearance is not an LLM verdict.** The audit's *pass* must come from the CI tests + a human
  domain reviewer confirming the external cohort is genuinely independent (a judgment the code cannot
  make). Claude drafts checks; it does not certify the benchmark leak-free.
- **Licence / access-tier / de-identification calls are human-verified.** Claude may *draft* the gate
  record, but `accessTier`, `permitsDerivedRedistribution`, and the de-id sign-off are decided by the
  licence+de-id reviewer. Controlled-vs-open is never "best-guessed" by the model.
- **Subtype/endpoint definitions and metric-appropriateness need the domain reviewer.** Claude can
  surface the candidate definition + citation; correctness and currency are the biostatistician's call.
- **No medical advice / no clinical interpretation.** Any patient-facing layer stays high-risk, gated
  on oncologist + advocate; Claude does not generate clinical guidance.

---

## 6. Ten concrete optimizations

1. **Make IBS + survival calibration the primary headline metric**, C-index secondary, and document
   that C-index is improper for t-year risks (Hartman 2023). Add bootstrap CIs to *every* reported
   number and to the optimism gap; forbid ranking models whose CIs overlap.
2. **Mandate nested/outer-held-out CV for both task families** (not just survival) in the methodology
   spec, and pin truncation time **tau** + IPCW scheme in `metricSpec`.
3. **Add three mandatory control baselines** — clinical/covariate-only, **label-shuffle (positive
   control for leakage)**, and **batch/centre-only** — and make a high-scoring label-shuffle baseline a
   *build-failing* leakage signal.
4. **Add a final blinded holdout cohort never exposed to the leaderboard**, plus per-team submission
   limits and an FDR/randomization-test correction on leaderboard rankings, to defeat multiple-comparison
   leaderboard overfitting.
5. **Require ≥2 external cohorts or leave-one-cohort-out** where data permits, so the optimism gap is
   not an artifact of one idiosyncratic validation set.
6. **Strengthen the batch-confound check** to a *predictive* test: can the batch variable predict the
   label as well as the features? Define where harmonization (ComBat) is fit — train-fold only — and
   add a "harmonization-leak" check.
7. **Forbid circular labels:** add a `labelSpec.derivation` field; if the subtype label was defined on
   the same cohort used as features, require an independently-labelled external cohort or flag the
   benchmark as "label-circular, discrimination-only."
8. **Pin the GDC data-release version and cBioPortal study version** in the provenance triple (not just
   URL+SHA), and define an explicit policy for source corrections/withdrawals (re-run + annotate +
   version-bump the leaderboard row).
9. **Resolve reproduction tolerance in M0** (per-metric absolute delta accounting for BLAS/GPU
   nondeterminism) since Definition-of-Shipped depends on it; ship a determinism smoke-test.
10. **Add resampling (SMOTE/over/under-sampling) and target-encoding to the preprocessing-isolation
    check** as fit-on-train-only transforms, and ship a `make reproduce` one-command runner that emits
    the full provenance bundle.

---

## 7. Parallel & perpendicular spin-offs

- **Generalized leakage-aware benchmark harness (domain-agnostic).** The audit suite + definition-object
  + provenance triple is valuable far beyond oncology (any biomedical-ML benchmark). Extract a
  `leakage-audit` core package; oncology becomes the first consumer. Natural OpenML-suite complement.
- **MCP server: "leakage-audit-as-a-service."** Expose the patient-overlap / preprocessing-isolation /
  batch-confound checks as MCP tools any agent (or reviewer) can call on a candidate split+pipeline,
  returning a pass/fail + taxonomy-mapped report. High leverage, low surface.
- **Provenance-gated public leaderboard** (the M3 artifact) as a standalone product: every row carries
  code+data+split provenance and an external-cohort number — a "leaderboard that can't lie."
- **`model-cards-oncology`** — shared model-card/datasheet generator and schema; directly reused here
  and a clean sibling deliverable.
- **`ewing-expression-reanalysis` / `proteomics-reanalysis`** — these reanalysis projects are *ideal
  first external cohorts or benchmark contributors*; the leakage harness audits their splits, and
  their honest re-runs feed the optimism-gap corpus. Tight, mutually-reinforcing tie.
- **`synthetic-cancer-data`** — synthetic cohorts as **leakage-audit test fixtures with known ground
  truth** (inject a known leak, confirm the audit catches it) and as a safe teaching dataset where the
  answer is known — strengthens the audit's own validation.
- **`pathology-image-benchmarks`** — the patient-overlap-leakage literature (~41% inflation) is
  strongest in histopathology; the same patient-grouped-split + site-confound machinery transfers
  directly. Deferred in this plan but the obvious perpendicular extension once the harness is general.
- **Teaching artifact: "How to evaluate oncology ML without leaking."** The worked
  positive/negative-control baselines + datasheets become a course module / reproducibility-workshop
  reference — a low-cost adoption channel and a hedge for the build-vs-mothball rule.

---

## 8. Open questions for the maintainer

1. **Primary metric stance:** will you adopt the §1.A recommendation to demote C-index to secondary in
   favor of IBS + calibration, given the "C-index is improper for t-year risk" result? This changes the
   leaderboard's headline column.
2. **Multiple-comparison defense:** is there appetite for a *truly* blinded final holdout + submission
   limits + ranking-stability correction, or is the single declared external cohort the intended
   ceiling? (This is the difference between "honest" and "looks honest.")
3. **Circular-label policy:** for the first subtyping benchmark, can you name an external cohort whose
   subtype label was derived *independently* of the TCGA training label, or will v1 be explicitly
   "discrimination-only, label-circular"?
4. **NC sources (COSMIC/OncoKB):** confirm the default lean to *exclude* so every artifact stays cleanly
   CC-BY/MIT — accepting that some benchmarks may be less rich — vs. accept-with-inherited-NC?
5. **Adopter/steward identity:** which is the realistic first adopter — a DREAM/Synapse-style
   reproducibility group, a journal's reproducibility-review process, or a teaching venue? This decides
   whether the leaderboard or the teaching artifact is built first, and it is the true critical path.
6. **Reproduction tolerance number:** what per-metric absolute delta (e.g. ±0.01 C-index/AUROC) counts
   as "reproduced," and is GPU nondeterminism in-scope or are baselines CPU-pinned for determinism?
7. **Harness generalization timing:** extract the domain-agnostic leakage-audit core (and MCP server)
   early (more reuse, more upfront design) or after M2 proves it on two oncology tasks?

---

### Source list (cited above)
- Kapoor & Narayanan, leakage & reproducibility crisis — https://www.sciencedirect.com/science/article/pii/S2666389923001599
- Data leakage in cancer drug-response prediction (2026) — https://www.biorxiv.org/content/10.64898/2026.02.05.704016v1.full
- Hartman et al., pitfalls of the concordance index (*Stat Med* 2023) — https://onlinelibrary.wiley.com/doi/full/10.1002/sim.9717
- Practical perspective on the concordance index — https://www.sciencedirect.com/science/article/pii/S1532046420301246
- scikit-survival, evaluating survival models — https://scikit-survival.readthedocs.io/en/stable/user_guide/evaluating-survival-models.html
- SurvivalEVAL — https://ojs.aaai.org/index.php/AAAI-SS/article/download/27713/27486/31764
- TorchSurv / pycox KKBox — https://opensource.nibr.com/torchsurv/
- soda-inria survival-analysis-benchmark — https://github.com/soda-inria/survival-analysis-benchmark
- DREAM Breast Cancer Prognosis Challenge — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3897241/
- DREAM mCRPC prostate challenge — https://pubmed.ncbi.nlm.nih.gov/27864015/
- OpenML Benchmarking Suites — https://arxiv.org/abs/1708.03731 ; https://pmc.ncbi.nlm.nih.gov/articles/PMC12416095/
- cBioPortal — https://pmc.ncbi.nlm.nih.gov/articles/PMC3956037/
- TCGA batch effects (germline/exome) — https://pmc.ncbi.nlm.nih.gov/articles/PMC6686424/
- Histology site-specific signatures (*Nat Commun* 2021) — https://www.nature.com/articles/s41467-021-24698-1
- Multimodal DL: prognostication or batch effect? (*Cancer Cell* 2022) — https://www.cell.com/cancer-cell/fulltext/S1535-6108(22)00522-0
- Radiomics reproducibility critique — https://www.dirjournal.org/articles/reproducibility-and-interpretability-in-radiomics-a-critical-assessment/doi/dir.2024.242719
- Reproducibility in ML for medical imaging — https://arxiv.org/pdf/2209.05097
- Histopath patient-overlap leakage (~41% inflation) — https://sjhatfield.github.io/Kaggle-melanoma-contest-exploration/
- PANDA-PLUS-Bench (prostate pathology robustness) — https://arxiv.org/pdf/2512.14922
