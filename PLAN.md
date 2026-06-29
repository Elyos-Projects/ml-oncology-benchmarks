# PLAN — ml-oncology-benchmarks

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## Executive summary

Machine-learning results in oncology are widely irreproducible and routinely **over-optimistic**.
The dominant cause is not weak models but **data leakage**: feature selection or normalization
performed on the full dataset before cross-validation, sample-level splits that put multiple
specimens from the *same patient* on both sides of a train/test boundary, batch/centre effects that
let a model "predict" the lab rather than the biology, gene signatures trained and reported on the
same cohort, duplicate or overlapping samples across "independent" cohorts, and survival models
scored with metrics that ignore censoring. The published "AUC 0.99 subtype classifier" or "C-index
0.85 survival model" frequently collapses on a genuinely external cohort. This costs the field
years of wasted follow-on work and erodes trust in computational oncology.

This project builds a small set of **documented, leakage-aware ML benchmarks** for two canonical
oncology tasks — **survival prediction** (overall / progression-free survival, censoring-aware) and
**molecular subtyping** (e.g. expression-based subtype classification) — built **only on
open-access, aggregate, or de-identified cancer data** with verified per-source licences. The
deliverable is not a model and not a clinical tool: it is a **reproducible benchmark** —
patient-grouped fixed splits, an explicit held-out *external validation cohort*, a leakage-audit
test suite that fails CI on any boundary violation, censoring-correct metrics, baseline
implementations, dataset **datasheets**, and per-model **model cards** carrying provenance on every
reported number. The benchmark exists to make honest evaluation the path of least resistance.

The **central design principle is the data/licensing-and-leakage gate**: no dataset enters a
benchmark until its licence is verified to permit research reuse and redistribution-of-derived-
metrics, its access tier is confirmed **open / aggregate / de-identified** (controlled-access data
is excluded by hard rule), and the benchmark's split + preprocessing design has passed an automated
leakage audit. This is built **first**, before any model is trained.

Risk tier is **medium** for the core research benchmark (domain accuracy, licence, and
de-identification correctness). Any **patient-facing** explainer ("what a survival model can and
cannot tell you") is **high** risk, carries a **"not medical advice"** label, and is **blocked on
oncologist *and* patient-advocate sign-off** — it is out of the core scope and gated as an optional,
expert-reviewed layer. No partner lab, data steward, or expert reviewer is yet secured, so all
tasks carry `verifiedNeed: false` and `requestor: TO BE SECURED` until confirmed.

## Problem & beneficiaries

**Who is helped.**
- **Computational-oncology / ML-method researchers** — get a credible, leakage-controlled yardstick
  to evaluate new survival/subtyping methods against honest baselines instead of self-reported,
  leaky numbers.
- **Reviewers, editors, and reproducibility groups** — get a reference standard and a re-runnable
  harness to check claims against.
- **Students and newcomers to cancer-data science** — get a worked, documented example of how to
  split, preprocess, and score oncology ML *without* leaking, which is rarely taught explicitly.
- **The public / patients (indirect).** The ultimate beneficiary is the patient: methods that are
  evaluated honestly are less likely to waste research effort or be over-claimed. We are explicit
  that this benefit is **indirect** — this project ships no clinical tool and gives no medical
  advice.

**The verified need.** The *general* problem — leakage and irreproducibility in biomedical ML — is
well established in the methodological literature (e.g. Kapoor & Narayanan's leakage taxonomy;
repeated demonstrations of batch-effect and patient-overlap leakage in genomic classifiers; the
MAQC/SEQC cautions on signature reproducibility). We treat the general need as **well established**.
The **specific, partner-anchored need is TO BE SECURED**: we have not yet confirmed a named lab,
benchmarking consortium, or data commons that will *adopt and use* the benchmark as their reference.
Until a concrete adopter/steward confirms, tasks carry `verifiedNeed: false`, because the Elyos bar
is "delivered, not merged" — a benchmark nobody adopts is not a delivered deed.

**Partner org.** TO BE SECURED. Candidate channels: an academic computational-oncology lab; a
reproducibility / open-science group; a cancer data-commons community (e.g. groups around the NCI
Genomic Data Commons, cBioPortal, or Synapse/DREAM-Challenge organisers); an ML-benchmark body. M0
includes explicit partner-and-expert outreach; no partner is assumed.

## Goals and non-goals

**Goals**
- Define and publish a **leakage-aware benchmark methodology specification** (split policy,
  preprocessing-isolation rules, metric definitions, external-validation requirement) that every
  benchmark in this project must implement and be tested against.
- Ship **≥ 2 reference benchmarks** — at least one **subtyping** and one **survival** task — each
  with: fixed patient-grouped splits, a genuinely external validation cohort, baseline models,
  censoring-/class-correct metrics, a dataset datasheet, and model cards with full provenance.
- Provide an **automated leakage-audit test suite** wired into CI that fails the build on
  patient-overlap, preprocessing-across-split, label/feature contamination, or duplicate-sample
  leakage.
- Make **licence + access-tier + de-identification verification** a non-skippable, auditable gate
  before any dataset is used.
- Provide a **task-agnostic, reproducible harness** (pinned environment, deterministic seeds,
  one-command re-run) so any third party can reproduce every reported number.
- Provide a **provenance-gated submission/leaderboard protocol** so external results are
  re-runnable and traceable to a code + data version, not just a claimed score.

**Non-goals**
- We do **not** build, deploy, or recommend a clinical/diagnostic/prognostic tool, and give **no
  medical advice**.
- We do **not** use controlled-access data (dbGaP, EGA, individual-level biobanks), nor *any*
  identifiable patient data, under any circumstance (see Data & compliance).
- We do **not** attempt to produce a "state-of-the-art" model or chase leaderboard wins; baselines
  exist to make the benchmark honest, not to compete.
- We do **not** re-identify, link, or re-aggregate de-identified records to individuals.
- We do **not** republish or mirror controlled or restrictively-licensed raw data; we redistribute
  only derived metrics/splits/metadata that the source licence permits.
- We do **not** ship patient-facing clinical content in the core scope; an education-only layer is
  optional, high-risk, and expert-gated (oncologist **and** advocate).

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics ("models trained", "datasets touched") are
explicitly excluded.

| Metric | Baseline | Target (first ~9 months) |
| --- | --- | --- |
| Reference benchmarks published with leakage audit passing + external validation cohort | 0 | ≥ 2 (≥ 1 subtyping, ≥ 1 survival) |
| Independent reproductions of a published benchmark number (third party re-runs harness, matches within tolerance) | 0 | ≥ 2 verified external reproductions |
| Adoption: external method evaluated against the benchmark by someone outside the project (cited, PR, or leaderboard submission that passes the re-run gate) | 0 | ≥ 3 verified external uses |
| Leakage caught: documented optimism gap (internal CV vs. external cohort) quantified and published per benchmark | n/a | Reported for 100% of benchmarks (the headline honesty output) |
| Datasets passing licence + access-tier + de-identification gate / datasets triaged-in | n/a | 100% of in-scope datasets carry a recorded, verifiable open/aggregate/de-id licence record |
| Confirmed adopter/steward + expert reviewer secured | 0 | ≥ 1 adopter/steward and ≥ 1 domain reviewer secured |
| Provenance coverage: reported numbers traceable to a (code version, data version, split hash) triple | n/a | 100% of leaderboard/model-card numbers |

Notes on attribution: an "external use" or "reproduction" must be **externally verifiable** (a
citation, a merged PR, a leaderboard submission whose re-run passes the gate). Self-reported use
does not count. The **optimism gap** (the drop from internal cross-validation to external-cohort
performance) is the project's signature outcome: surfacing it honestly is the public good, even — and
especially — when it is large.

## Scope

**In scope**
- The **leakage-aware benchmark methodology spec** (split, preprocessing isolation, metrics,
  external-validation requirement, reporting template).
- **Two task families:** (a) **molecular subtyping** — multi-class classification of an established
  subtype label from open expression (or proteomic) features; (b) **survival prediction** —
  overall / progression-free survival with right-censoring, scored by concordance and
  censoring-aware metrics.
- **Datasets:** open-access / aggregate / de-identified only (see Data section). Concretely the
  candidate pool is open-tier TCGA-derived expression + de-identified clinical (via the GDC open
  tier and/or cBioPortal), GEO series, CPTAC open proteogenomics, and aggregate registry statistics
  (SEER\*Stat aggregate, GLOBOCAN) where a benchmark uses population-level data only.
- **Harness:** pinned reproducible environment, deterministic splits, baselines, metric library,
  leakage-audit suite, model-card + datasheet generators, provenance ledger.
- **Submission/leaderboard protocol** with a provenance + reproducibility re-run gate.

**Out of scope**
- **Controlled-access data** (dbGaP, EGA, individual-level biobanks, any access requiring a DUA/IRB)
  and **any identifiable patient data** — excluded by hard rule, never "best-guessed."
- Any **clinical, diagnostic, prognostic, or treatment** tool or recommendation; any **medical
  advice**.
- Re-identification, record linkage to individuals, or re-derivation of individual-level data from
  aggregates.
- Imaging benchmarks on whole-slide / radiology images in the first phases (large, separate licence
  + de-id surface; deferred to backlog and only if a clean open-licence path exists).
- "Winning" the task (SOTA chase); productionising or serving any model.
- Patient-facing clinical content in core scope (optional, high-risk, expert-gated layer only).

## Solution approach & architecture

This is a **research/data + light-software** project: the durable artifacts are the **methodology
spec, the fixed benchmark definitions, the datasheets/model cards, and the audit + harness code**.

**Benchmark definition (the unit of work).** Each benchmark is a self-contained, versioned
definition object — the single source of truth — containing:
`id`, `task` (`subtyping` | `survival`), `cancerType`, `sources[]` (each with licence + access-tier
+ provenance record), `featureSpec` (modality, gene/feature set, provenance of the feature list),
`labelSpec` (subtype definition + citation, or survival endpoint + censoring definition),
`splitSpec` (grouping key = patient ID, fold scheme, external-cohort identity, split hashes),
`preprocessingSpec` (fit-on-train-only transforms), `metricSpec` (primary + secondary metrics),
`baselines[]`, `version`, and `leakageAuditResult`. All outputs (leaderboard rows, model cards,
reports) are **projections** of this object.

**Pipeline (per benchmark).**
1. **Triage & gate** — identify dataset(s), publisher, licence, access tier, and de-identification
   status. PASS only if open/aggregate/de-identified **and** licence permits research reuse +
   redistribution of derived metrics/splits. Record the decision artifact. (Blocking.)
2. **Datasheet** — Datasheets-for-Datasets questionnaire per source (motivation, composition,
   collection, de-identification, licence, known biases, batch/centre structure).
3. **Split design** — patient-grouped splits with a **declared external validation cohort** from a
   *different* study/centre wherever possible; emit split-membership files + content hashes.
4. **Leakage audit** — run the automated audit (below) over the split + preprocessing plan; a
   failure blocks the benchmark.
5. **Baselines + metrics** — train baselines fit-on-train-only; score with censoring-/class-correct
   metrics; compute the **internal-CV vs. external-cohort optimism gap**.
6. **Model cards + provenance** — per-model card with the (code version, data version, split hash)
   triple and every reported number cited to its computation.
7. **Reproducibility re-run** — independent one-command re-run reproduces numbers within tolerance.
8. **Review & publish** — domain reviewer + technical reviewer sign off; results published with the
   benchmark definition and audit artifact.

**The leakage-audit suite (the heart of the project).** A set of programmatic checks, each a CI test
that fails the build on violation:
- **Patient-overlap check** — no patient/case ID (or known sample-alias) appears in more than one
  split; cross-cohort overlap (same patient in "internal" and "external" cohort) is detected and
  rejected.
- **Preprocessing-isolation check** — all fitted transforms (scaling, imputation, feature selection,
  PCA, normalization, batch correction) are fit on training folds **only**; static analysis +
  runtime assertion that no test data is seen during `fit`.
- **Feature/label-contamination check** — labels are not derivable from features by construction
  (e.g. a survival feature that encodes the outcome, a subtype "marker" that is the label proxy);
  flags near-duplicate feature/label leakage and target encoding done across the split.
- **Duplicate/near-duplicate sample check** — detect duplicated or technically-replicated profiles
  across splits (correlation/fingerprint threshold).
- **Batch/centre confounding check** — test whether the split or the label is confounded with a
  detectable batch/centre/platform variable; report the confound and require it be addressed or
  declared.
- **Temporal / lookahead check** (survival) — no post-baseline or outcome-derived feature is used as
  a predictor; censoring handled correctly.
- **Metric-correctness check** — survival scored with censoring-aware metrics (C-index/Harrell's &
  Uno's, time-dependent AUC, integrated Brier score), classification with class-imbalance-aware
  metrics (balanced accuracy, macro-F1, AUROC/AUPRC) and proper calibration where claimed.

**Tech stack.** Per Elyos conventions for tooling/CI orchestration: TypeScript/ESM + pnpm for the
harness CLI, provenance ledger, datasheet/model-card generators, and the orchestration of audits;
**Python** for the actual ML/statistics (scikit-learn, lifelines/scikit-survival, pandas/numpy) — ML
in oncology is a Python ecosystem and forcing it into TS would be dishonest. The TS layer drives,
records, and validates; the Python layer computes, behind a pinned, containerised environment. All
randomness seeded; environments pinned (lockfiles + container digest) so re-runs are deterministic.
No runtime service; everything runs locally or in CI.

**Key decisions.**
- **Definition-object-first** so leaderboard, model cards, and reports are projections, never
  hand-maintained parallels.
- **Leakage audit is blocking**, encoded as committed CI tests, not a checklist someone signs.
- **External validation cohort is mandatory** for any performance claim; internal CV alone is never
  a headline number.
- **Redistribute derived artifacts only** (metrics, split hashes, feature lists, metadata) — never
  re-host source data unless the source licence explicitly permits and access tier is open.
- **Python-for-ML / TS-for-orchestration** split is deliberate and documented, not vendor lock-in.

## Data, licensing & compliance

**This is the critical, conservative section — the cancer guardrails lead here.**

### Binding cancer guardrails (non-negotiable)
- **Open-access / aggregate / de-identified data ONLY.** Controlled-access data — **dbGaP, EGA,
  individual-level biobanks**, or anything requiring a Data Use Agreement, IRB approval, or
  authorized credentials — is **OUT OF SCOPE** and excluded by hard rule. **Any identifiable patient
  data is out of scope.**
- **No medical advice.** The benchmark and all artifacts are research methodology, not clinical
  guidance. Any patient-facing explainer is education only, carries a prominent **"not medical
  advice"** notice, and is **blocked on oncologist + patient-advocate review** (riskTier `high`).
- **Provenance on every assertion.** Every reported number, every subtype/endpoint definition, every
  dataset claim is traceable to a cited source + a (code, data, split) version triple. "No source,
  no claim."

### Per-source licence & access-tier verification (must pass before use)
Each source is verified and recorded *before* any modelling. Indicative classification (each entry
is re-verified per dataset at gate time, never assumed):
- **TCGA (via NCI GDC open tier / cBioPortal):** open-access **clinical, de-identified, and
  derived/processed expression** data are usable under the **NIH Genomic Data Sharing** open-access
  terms — **no DUA** for the open tier; publication-attribution expected. **Controlled-tier** GDC
  data (raw sequence/genotype) is **excluded**. Use only the open tier.
- **GEO (NCBI Gene Expression Omnibus):** public-access series; redistribution generally permitted,
  per-series terms recorded. Treated as usable for derived metrics; provenance + accession recorded.
- **CPTAC (open proteogenomics):** open-access tier usable; controlled tiers excluded.
- **PRIDE (proteomics):** typically **CC-BY / CC0** per dataset — accepted with attribution;
  per-accession licence recorded.
- **SEER / GLOBOCAN:** **aggregate** incidence/survival statistics (e.g. SEER\*Stat aggregate
  outputs, GLOBOCAN) are usable population-level data; **individual-level SEER research data
  (DUA-gated) is OUT OF SCOPE** — only aggregate outputs are used.
- **COSMIC:** **non-commercial licence** — usable only under its NC terms for non-commercial public
  benefit, attribution recorded; flagged so downstream reusers know the restriction. Treated as
  **restricted**, not freely-open.
- **OncoKB:** **licence required / non-commercial** — only used if its terms permit the specific
  reuse; otherwise excluded. Treated as **restricted**.
- **Anything else** (custom terms, "all rights reserved", unclear redistribution) → **FLAG/EXCLUDE**,
  never default-allow.

**Objective gate criterion.** A dataset PASSes only if **all** are recorded with cited evidence:
`accessTier: open|aggregate|deidentified` (controlled ⇒ EXCLUDE), `licenceId` + `licenceUrl` on the
accepted/restricted list, `permitsResearchReuse: true`, `permitsDerivedRedistribution: true`
(derived metrics/splits/metadata), `deidentification: publisher-documented`, and an attribution
string. A `restricted` (NC) source may still PASS for **non-commercial** benchmark use but is
labelled so all derived artifacts carry the inherited restriction. Missing evidence on any field =
EXCLUDE.

**De-identification stance.** We use only data the **publisher** has already de-identified to its
documented standard; we never de-identify, re-identify, or attempt linkage ourselves. If any field
in an open dataset looks individually identifying or finer-grained than the publisher's stated
aggregation (e.g. exact dates, rare-disease + small-cell-size + geography combinations enabling
re-identification), the dataset/field is **flagged and excluded** and the concern surfaced. We do
not redistribute row-level data; benchmark artifacts are **split memberships (by opaque case ID),
derived features as permitted, and metrics**.

**Provenance model.** Every source records: accession/URL, publisher, retrieval timestamp,
dataset/version, access tier, licence id + URL + a captured snapshot (committed copy + SHA-256 hash +
Wayback save URL), de-identification note, batch/centre structure note, and attribution string.
Every benchmark records the (code version, data version, split hash) triple for each number.
Provenance is part of the committed deliverable.

**Attribution & output licences.** All sources attributed per their licence. **Benchmark code +
audit suite + harness: MIT.** **Datasheets, model cards, methodology spec, reports, leaderboard:
CC-BY-4.0.** Any artifact derived from an **NC source (COSMIC/OncoKB)** inherits and displays the
**non-commercial** restriction (so it is *not* relicensed permissively).

## Quality, review & risk gates

**Risk tier: medium** for the core research benchmark (domain accuracy, licence, de-identification,
statistical correctness). **High** for any patient-facing explainer (gated, optional, out of core
scope).

**Required review before a deed is "done":**
- **Licence + de-identification reviewer** (mandatory, every dataset): confirms open/aggregate/de-id
  access tier, verified licence, and absence of identifiable/re-identifiable data. **Hard gate** —
  no benchmark uses a dataset without this sign-off. Must be filled **before the M1 pilot benchmark
  is reviewed**.
- **Domain reviewer (oncology data-science / biostatistics)** (mandatory, every benchmark): confirms
  the subtype/endpoint definitions are correct and current, the metrics are appropriate, censoring
  is handled correctly, and the leakage controls are biologically meaningful (e.g. that the declared
  "external" cohort is genuinely independent). Medium-tier sign-off.
- **Technical reviewer:** confirms the harness, audit suite, baselines, and reproducibility re-run
  are correct and CI-green.
- **Oncologist + patient-advocate (HIGH-tier, blocking)** — **required and blocking** for *any*
  patient-facing/education artifact before it ships; verifies "not medical advice" framing and
  absence of clinical recommendation. No patient-facing artifact ships without **both** sign-offs.

**Leakage-audit gate.** Every benchmark must pass the full leakage-audit suite in CI. A failing
audit blocks publication. The audit result is committed as part of the benchmark definition.

**Definition of Shipped.** A benchmark is *shipped* when: (1) all its datasets passed the
licence/access/de-id gate with recorded provenance; (2) the leakage-audit suite passes in CI; (3) it
reports both internal-CV and **external-cohort** performance with the optimism gap quantified;
(4) every number is provenance-traced (code+data+split triple) in a model card; (5) an independent
one-command re-run reproduces the numbers within tolerance; (6) domain + technical + licence/de-id
reviewers have signed off; **and (7) it has been adopted/used by a real beneficiary outside the
project** (a lab/consortium/reproducibility group cites, submits to, or builds on it). Producing the
benchmark is *not* shipped; adoption + reproduction by a beneficiary is. *(Gated on a secured
adopter/steward — TO BE SECURED.)*

## Roadmap & milestones

Each phase has a goal and measurable **exit criteria**. M0 is a thin foundation/cold-start; later
phases scale. Sequencing respects that the methodology + gate must precede any modelling.

**M0 — Methodology, gate & skeleton (cold-start, thin)**
- Goal: write the leakage-aware methodology spec and the licence/access/de-id gate, stand up the
  repo + CI + harness skeleton, and triage one candidate dataset through the gate — before any model
  is trained. Begin adopter/expert outreach.
- Exit criteria: (1) **leakage-aware benchmark methodology spec** published and reviewed (split,
  preprocessing-isolation, metrics, external-validation requirement, reporting template); (2)
  **licence + access-tier + de-identification gate** checklist exists and is applied to ≥ 1 dataset
  with a committed gate artifact; (3) repo + pnpm/TS + pinned Python container + CI (build/test/lint)
  green; (4) the **leakage-audit suite skeleton** runs in CI with golden pass/fail fixtures for the
  patient-overlap and preprocessing-isolation checks; (5) one candidate dataset has a published
  **datasheet**; (6) ≥ 1 adopter/expert outreach thread opened.

**M1 — Reference subtyping benchmark v1 (one task, leakage-aware, external cohort)**
- Goal: deliver one complete subtyping benchmark end-to-end on open data, with an external
  validation cohort and a passing leakage audit.
- Exit criteria: (1) one subtyping benchmark fully defined (definition object + datasheet + splits)
  on gated-in open data; (2) full leakage-audit suite passes in CI; (3) baselines trained
  fit-on-train-only; class-correct metrics reported; (4) **external-cohort** result reported and the
  **optimism gap** quantified; (5) per-model model cards with the code+data+split provenance triple;
  (6) independent one-command re-run reproduces numbers within tolerance; (7) domain + licence/de-id
  + technical sign-offs recorded.

**M2 — Survival benchmark + task-agnostic harness**
- Goal: add a survival benchmark (censoring-correct) and generalise the harness so a new benchmark
  is a definition object, not a fork.
- Exit criteria: (1) one survival benchmark with right-censoring, nested CV, external cohort, and
  censoring-aware metrics (Harrell + Uno C-index, time-dependent AUC, integrated Brier score);
  (2) temporal/lookahead leakage check active in CI; (3) harness refactored to run both task
  families from definition objects with shared audit/metric/provenance code; (4) optimism gap
  reported; (5) all M1 sign-offs repeated for the survival benchmark.

**M3 — Submission protocol, leaderboard & multi-dataset breadth**
- Goal: let external parties submit results reproducibly; add breadth across ≥ 2 cancer types or
  cohorts; publish a provenance-gated leaderboard.
- Exit criteria: (1) submission protocol + **reproducibility re-run gate** (a submission is accepted
  only if its numbers re-run within tolerance and its data is in-scope/gated); (2) provenance-gated
  leaderboard live (every row carries code+data+split provenance and an external-cohort number);
  (3) ≥ 2 benchmarks across ≥ 2 cancer types/cohorts; (4) ≥ 1 external submission passes the re-run
  gate.

**M4 — Adoption, reproduction & sustainability (the deed)**
- Goal: secure a real adopter/steward, get independent reproductions, and stand up maintenance.
  Optionally publish the expert-gated education layer.
- Exit criteria: (1) adopter/steward secured (`verifiedNeed` flips true); (2) ≥ 2 verified external
  reproductions and ≥ 3 verified external uses; (3) maintenance rotation + data/licence
  re-verification cadence documented; (4) **(optional)** patient-facing education explainer
  published **only** with oncologist **and** advocate sign-off + "not medical advice" label, or
  explicitly deferred. If no adopter is secured by the dated decision point, apply the
  **build-vs-mothball/pivot rule** (hand the benchmark to a teaching/reproducibility venue as a
  reference deed, or mothball to maintenance-only) rather than declare a deed with no beneficiary.

Dependencies: M1 depends on M0's spec + gate + audit skeleton; M2 reuses M1's harness; M3 depends on
≥ 1 stable benchmark; M4 depends on a body of reproducible benchmarks + a secured adopter.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above. Each
milestone has a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on |
Reviewer`), acceptance criteria for the most important tasks, and a milestone Definition of Done.
`TASKS.md` also carries a backlog of sized-but-unscheduled tasks and one complete, schema-valid
example Task JSON for the first M0 task. New datasets enter as per-dataset gate+datasheet tasks; new
benchmarks enter as definition-object tasks that must pass the leakage audit before modelling.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the methodology spec, harness, audit suite, and backlog.
- **Licence + de-identification reviewer:** TBD (TO BE SECURED) — mandatory, **non-skippable**
  gatekeeper for access tier, licence, and de-identification; no benchmark uses a dataset without
  this sign-off. Must be filled before the M1 pilot is reviewed. May rotate among ≥ 2 qualified
  reviewers, but ≥ 1 must always exist or triage halts.
- **Domain reviewer (oncology data-science / biostatistics):** TBD (TO BE SECURED) — mandatory per
  benchmark; verifies subtype/endpoint definitions, metric appropriateness, censoring handling, and
  that "external" cohorts are genuinely independent (medium-tier sign-off).
- **Technical reviewer(s):** rotation of contributors verifying harness/audit/baselines/re-run (CI
  green).
- **Oncologist + patient-advocate reviewers (HIGH tier):** TBD (TO BE SECURED) — **blocking** for any
  patient-facing/education artifact; both required.
- **Steward (last-mile owner / adopter liaison):** TBD (TO BE SECURED) — owns the relationship with
  the adopting lab/consortium and records adoption + reproduction events (the "delivered" signal).
- **Partner / requestor:** TO BE SECURED — named adopting lab/consortium/data-commons community.

## Dependencies & integrations

- **Open data sources (per-source gated):** NCI GDC open tier / cBioPortal (TCGA-derived open
  clinical + expression), GEO, CPTAC open tier, PRIDE, aggregate SEER\*Stat / GLOBOCAN. Restricted
  (NC) sources COSMIC/OncoKB only under their terms and only if needed. **No controlled-access
  sources.**
- **External standards/specs (pinned):** Datasheets for Datasets (Gebru et al.); Model Cards
  (Mitchell et al.); TRIPOD / TRIPOD-AI reporting guidance for prognostic models; reporting
  conventions from MAQC/SEQC and the ML-leakage taxonomy (Kapoor & Narayanan) for the audit design;
  SPDX licence identifiers. Versions recorded and bumped only via a deliberate task.
- **Libraries (pinned, containerised):** scikit-learn, scikit-survival, lifelines, pandas, numpy
  (Python); the TS/pnpm harness for orchestration, provenance, generators, and CI.
- **Elyos pieces:** Task JSON schema (`packages/schema`), the donated-lane CLI workspace/PR flow
  (`packages/cli`), the good-deed definition + refusal guardrails. No funded-lane/runner dependency
  (donated lane); any funded compute task would declare a `fundedBudgetUsd` cap.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Accidentally using controlled-access or identifiable data | Low | Critical | Hard access-tier gate before any use; open/aggregate/de-id only; exclude on any doubt; reviewer sign-off recorded | Licence + de-id reviewer |
| Subtle leakage survives the audit (the project's own failure mode) | Medium | High | Multi-check audit in CI; mandatory external cohort; domain reviewer checks cohort independence; publish optimism gap; treat any discovered leak as a regression test added within one release | Maintainer + Domain reviewer |
| Misclassifying a licence (treating NC/restricted as open) | Medium | High | Per-source licence record with cited clause + snapshot; NC sources (COSMIC/OncoKB) labelled and restriction inherited by derivatives; exclude on doubt | Licence + de-id reviewer |
| "External" cohort is not truly independent (shared patients/batches) | Medium | High | Patient-overlap + duplicate-sample + batch-confound checks; domain reviewer confirms provenance independence | Domain reviewer |
| Non-reproducible numbers (env drift, unseeded randomness) | Medium | Medium | Pinned container + lockfiles + seeds; one-command re-run gate; reproduction is a Definition-of-Shipped criterion | Technical reviewer |
| No adopter secured → benchmark built but unused (fails "delivered") | Medium | High | M0 outreach; steward role; `verifiedNeed:false` until secured; dated build-vs-mothball/pivot rule | Steward |
| Benchmark misread as a clinical/prognostic tool | Medium | High | Prominent "research only, not medical advice"; no clinical recommendation; patient-facing layer gated behind oncologist+advocate | Maintainer |
| Source data changes/withdrawn, breaking a benchmark | Medium | Medium | Version + retrieval date + snapshot recorded; harness detects data drift; maintenance milestone | Maintainer |
| Scope creep into SOTA-chasing or productionising a model | Medium | Medium | Explicit non-goal; reviewers reject; baselines are reference-only | Maintainer |
| Optimism-gap / negative results discourage contributors | Low | Medium | Frame honest evaluation as the product; celebrate leakage caught as a win | Maintainer |

## Security & privacy

- **Threat surface is small** (no runtime service; no hosting of source data). Main surfaces are CI,
  the harness, and the derived artifacts we publish.
- **Data minimisation.** We use only open/aggregate/de-identified data and redistribute only derived
  metrics, split memberships (opaque IDs), permitted derived features, and metadata — never row-level
  identifiable data, never controlled data.
- **No re-identification.** Re-identification, linkage to individuals, or re-deriving individual data
  from aggregates is forbidden and flagged as a refusal.
- **Secrets handling.** No credentials needed for open sources. If any source ever requires a token,
  it is supplied by the human, never written to logs, receipts, model cards, leaderboard, or
  committed files (per Elyos rules). API keys for any funded compute are capped and never logged.
- **Abuse/misuse prevention.** Refuse and flag any task steering the work toward: clinical
  decision-making or medical advice; re-identification/linkage; laundering controlled or
  restrictively-licensed data as "open"; building surveillance or patient-targeting; or producing an
  over-claimed "diagnostic" model. The benchmark stays descriptive, reproducible, and source-cited.

## Sustainability & maintenance

- **Post-delivery ownership.** The steward maintains the adopter/consortium relationship and records
  adoption/reproduction outcomes; the maintainer keeps the spec, harness, audit suite, and pinned
  environment current.
- **Re-verification cadence.** Source licences and access tiers are re-verified on a fixed cadence
  (and on any source change); subtype/endpoint definitions are re-checked against current literature
  by the domain reviewer; stale benchmarks become `maintenance` tasks.
- **Outcome tracking.** The steward records external reproductions, external uses, and per-benchmark
  optimism gaps against the success metrics; reviewed each milestone. Negative/honest results are
  retained, not pruned.

## Open questions

- Which specific lab / consortium / data-commons community will be the first confirmed adopter and
  steward? (Gates `verifiedNeed`.)
- For the first subtyping benchmark, which **independent external cohort** gives genuine
  cross-study/cross-platform separation (e.g. a GEO series independent of the TCGA-derived training
  cohort) while staying fully open-access?
- Do we accept **COSMIC/OncoKB (NC)** sources at all, given the restriction inherited by every
  derivative, or exclude them to keep all outputs cleanly CC-BY/MIT? (Default lean: exclude unless a
  benchmark genuinely needs them and the steward accepts the inherited NC label.)
- What numeric **reproduction tolerance** counts as "reproduced" per metric (absolute C-index /
  AUROC delta), accounting for hardware-level nondeterminism?
- Where is compute run for the funded lane, if any benchmark is too large for a donated session, and
  what is the hard `fundedBudgetUsd` cap per benchmark?
- Do we ever ship the patient-facing education layer, or keep this project purely researcher-facing?

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8 cancer guardrails) — `C:\code\elyos\planning\ROADMAP.md`
- Exemplar plan (depth reference) — `C:\code\Ofelia\plan.md`
- Datasheets for Datasets (Gebru et al., 2018/2021)
- Model Cards for Model Reporting (Mitchell et al., 2019)
- TRIPOD / TRIPOD-AI reporting guidance for prognostic/diagnostic models
- "Leakage and the Reproducibility Crisis in ML-based Science" (Kapoor & Narayanan, 2023)
- MAQC/SEQC consortium guidance on gene-signature reproducibility
- NIH Genomic Data Sharing Policy; NCI GDC data-access tiers (open vs. controlled)
- cBioPortal, GEO, CPTAC, PRIDE, SEER\*Stat / GLOBOCAN access & licence terms
- COSMIC and OncoKB non-commercial licence terms
- SPDX licence list; Creative Commons CC-BY 4.0; MIT licence

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified during drafting and **applied** to this plan
and to `TASKS.md` (not left as suggestions):

1. **Hard access-tier gate added** — the gate now records an explicit `accessTier` (open/aggregate/
   de-identified) and *excludes controlled* (dbGaP/EGA/biobank) by rule, not by reviewer memory.
2. **Patient-grouped splitting made the default split unit** — splits group on patient/case ID, not
   sample, closing the most common oncology leakage (multiple specimens per patient across folds).
3. **External validation cohort made mandatory** for any headline performance number; internal CV
   alone is never a published claim.
4. **Optimism gap promoted to the signature outcome metric** — internal-CV vs external-cohort drop
   is reported for 100% of benchmarks, reframing "honest, even if disappointing" as the product.
5. **Leakage audit encoded as blocking CI tests** with golden pass/fail fixtures, rather than a
   checklist someone signs.
6. **Preprocessing-isolation check added** — static + runtime assertion that scalers/imputers/
   feature-selectors/PCA/batch-correctors are fit on train folds only.
7. **Batch/centre confounding check added** — surfaces the case where the model predicts the lab,
   not the biology, and requires it be addressed or declared.
8. **Censoring-correct survival metrics specified** — Harrell *and* Uno C-index, time-dependent
   AUC, integrated Brier score; naive accuracy/AUC on survival explicitly rejected.
9. **Provenance triple (code, data, split) required** on every reported number, making "no source,
   no claim" mechanically enforceable.
10. **Reproducibility re-run gate added** to Definition of Shipped and to the submission protocol —
    a number that doesn't re-run within tolerance is not accepted.
11. **Python-for-ML / TS-for-orchestration decision made explicit** and justified, instead of
    pretending the ML can be pure TypeScript.
12. **NC-source restriction inheritance specified** — artifacts derived from COSMIC/OncoKB carry the
    non-commercial label; they are not silently relicensed as CC-BY/MIT.
13. **De-identification stance hardened** — we never de-/re-identify; we use only publisher-de-id'd
    data and exclude finer-than-aggregation fields.
14. **SEER/individual-level explicitly excluded**, only aggregate SEER\*Stat/GLOBOCAN allowed,
    closing a subtle controlled-data trap.
15. **Duplicate/near-duplicate sample check added** to catch the same profile appearing in "internal"
    and "external" cohorts (a real cross-cohort leakage mode).
16. **Patient-facing layer demoted to optional + high-risk**, blocked on oncologist *and* advocate
    sign-off with a "not medical advice" label, keeping the core medium-risk.
17. **`verifiedNeed:false` everywhere until an adopter is secured**, with an honest "general need
    established, specific need TO BE SECURED" distinction.
18. **Build-vs-mothball/pivot decision rule added** so a benchmark with no adopter is pivoted or
    mothballed, not falsely declared a delivered deed.
19. **Definition-object-first architecture** so leaderboard/model-cards/reports are projections, not
    hand-maintained duplicates that drift.
20. **Licence-snapshot format fixed** (committed copy + SHA-256 + Wayback URL) so provenance is
    durable against source pages changing.
21. **TRIPOD-AI reporting guidance adopted** as the reporting template basis, aligning with the
    clinical-prediction-model reporting standard reviewers expect.
22. **Domain-reviewer duty to confirm external-cohort independence** added explicitly — the audit
    can't prove independence alone; a human verifies provenance.
23. **Outcome metrics made externally verifiable** (citation/merged-PR/passing-re-run only),
    excluding self-reported adoption.
24. **Refusal triggers enumerated** for this domain (clinical advice, re-identification, data
    laundering, surveillance, over-claimed diagnostics) in Security & privacy.
25. **Funded-lane budget-cap hook noted** (any oversized benchmark uses a hard `fundedBudgetUsd`
    cap, never uncapped compute) consistent with Elyos funded-lane rules.

---

## Review sign-off

**Reviewer:** drafting engineer (self-review pass for completeness + correctness).
**Date:** 2026-06-28. **Version reviewed:** 0.1.0.

**Completeness check.** All 17 required PLAN sections are present and in spec order (Executive
summary → References), followed by Appendix A and this sign-off. `TASKS.md` is present with the
schema-mapping section, milestone tables matching the M0–M4 roadmap, per-task acceptance criteria,
milestone Definitions of Done, a backlog, and a schema-valid example Task JSON.

**Correctness fixes applied during review.**
- Verified the example Task JSON against `packages/schema/src/schemas.ts`: all required fields
  present (`id, title, project, type, lane, priority, domain, riskTier, urgent, deliverable,
  tokenEstimate, status, context, objective, acceptanceCriteria, output, verifiedNeed`), all enum
  values legal, `acceptanceCriteria` non-empty, `output` non-empty, no `additionalProperties`. The
  example is `donated` lane so no `fundedBudgetUsd` is required.
- Confirmed the cancer guardrails lead the Data section and are marked binding; controlled-access and
  identifiable data are excluded by hard rule, not reviewer discretion.
- Confirmed the patient-facing layer is HIGH risk with a **blocking** oncologist + advocate gate and
  is out of core scope.
- Confirmed `verifiedNeed: false` and `requestor: TO BE SECURED` are consistent across PLAN and the
  example task (no partner secured yet).
- Confirmed NC sources (COSMIC/OncoKB) carry inherited restriction and are not relicensed.

**Residual risks accepted (tracked in Open questions).** First adopter/steward unidentified; the
specific independent external cohort for the first benchmark unselected; reproduction tolerance
unquantified; whether to admit NC sources at all undecided. These are honest unknowns, not
omissions, and each is owned in Open questions / Risks.

**Sign-off:** plan is internally consistent, schema-aligned, guardrail-compliant, and ready for
maintainer + (once secured) domain/licence/expert review.
