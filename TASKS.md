# ml-oncology-benchmarks — TASKS.md

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

Backlog for **ml-oncology-benchmarks** (slug: `ml-oncology-benchmarks`): documented, **leakage-aware**
open ML benchmarks for **survival** and **molecular subtyping** on **open-access / aggregate /
de-identified** cancer data. See `PLAN.md` for full context.

The **leakage-aware methodology spec + the licence/access-tier/de-identification gate are the first
build items** and hard product requirements. This is a **MEDIUM** risk-tier project on its core
research surface (domain accuracy, licence, de-identification, statistical correctness). Any
**patient-facing/education** task is **HIGH** risk and is **blocked on oncologist + patient-advocate
sign-off** with a "not medical advice" label. **Controlled-access data (dbGaP/EGA/biobanks) and any
identifiable patient data are OUT OF SCOPE.** No adopter/steward or expert reviewer is yet secured,
so all tasks carry `requestor: TO BE SECURED` and `verifiedNeed: false`.

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against `packages/schema/src/schemas.ts`.
Field mapping:

- **id** — stable slug `ml-oncology-benchmarks-<area>-NNN`.
- **title** — the task title in the milestone table.
- **project** — `ml-oncology-benchmarks`.
- **type** — one of `code | research | writing | data | design-spec | maintenance`.
- **lane** — `donated` (default; no funded tasks planned. Any `funded` task must add
  `fundedBudgetUsd` with a hard cap).
- **priority** — `high | medium | low`.
- **domain** — array, e.g. `["cancer","oncology","machine-learning","data","reproducibility"]`.
- **riskTier** — `low | medium | high`. Datasets/benchmarks touching de-identification, licence,
  subtype/endpoint correctness, or statistics are `medium`; pure skeleton/infra is `low`; any
  patient-facing/education artifact is `high`.
- **urgent** — boolean (no urgent tasks at cold-start).
- **deliverable** — `pr | dataset | document | translation`.
- **tokenEstimate** — `small | medium | large` (maps to the Size column).
- **status** — `open | in-progress | review | delivered | done` (all start `open`).
- **context / objective / acceptanceCriteria[] / resources[] / output** — per task.
- **requestor** — adopter/steward/expert; `TO BE SECURED` where unknown.
- **verifiedNeed** — `true` only once a specific adopter/need is confirmed; currently `false`
  everywhere (no adopter secured).
- **outputLicense** — code: `MIT`; datasheets/model-cards/spec/reports/leaderboard: `CC-BY-4.0`;
  any artifact derived from an NC source (COSMIC/OncoKB) inherits a **non-commercial** restriction.

Size legend: small ≈ tokenEstimate `small`, med ≈ `medium`, large ≈ `large`.
Reviewer key: **Maintainer**; **Licence+de-id reviewer** (access tier + licence + de-identification,
hard gate, TO BE SECURED); **Domain reviewer** (oncology data-science/biostatistics, TO BE SECURED);
**Technical reviewer**; **Oncologist + Advocate** (HIGH-tier blocking, TO BE SECURED).

---

## Milestone M0 — Methodology, gate & skeleton (cold-start)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ml-oncology-benchmarks-methodology-001 | Leakage-aware benchmark methodology specification (splits, preprocessing isolation, metrics, external-validation requirement, reporting template) | design-spec | medium | medium | document | — | Domain reviewer + Maintainer |
| ml-oncology-benchmarks-gate-002 | Licence + access-tier + de-identification gate checklist (open/aggregate/de-id only; controlled excluded) | design-spec | medium | medium | document | — | Licence+de-id reviewer + Maintainer |
| ml-oncology-benchmarks-repo-003 | Monorepo skeleton: pnpm/TS/ESM harness + pinned Python ML container + CI (build/test/lint) | code | small | low | pr | — | Maintainer |
| ml-oncology-benchmarks-audit-004 | Leakage-audit suite skeleton (patient-overlap + preprocessing-isolation checks) wired into CI with golden pass/fail fixtures | code | medium | medium | pr | 001, 003 | Technical reviewer + Domain reviewer |
| ml-oncology-benchmarks-datasheet-005 | Triage one candidate open dataset through the gate + publish its datasheet | data | medium | medium | dataset | 002 | Licence+de-id reviewer + Domain reviewer |
| ml-oncology-benchmarks-outreach-006 | Adopter/steward + expert-reviewer outreach (open ≥ 1 thread; record decision criteria + deadline) | research | small | low | document | — | Maintainer |

**Acceptance criteria — key tasks**

- **ml-oncology-benchmarks-methodology-001** (methodology spec)
  - Mandates **patient/case-grouped** splitting as the default unit (no patient on both sides of any
    split) and a **declared external validation cohort** from a different study/centre for any
    headline number; internal CV alone is never a published claim.
  - Specifies **preprocessing isolation**: all fitted transforms (scaling, imputation, feature
    selection, PCA, normalization, batch correction) fit on training folds only.
  - Defines task-correct metrics: **survival** = Harrell & Uno C-index, time-dependent AUC,
    integrated Brier score (censoring-aware); **subtyping** = balanced accuracy, macro-F1,
    AUROC/AUPRC, calibration; naive accuracy/AUC on censored data explicitly rejected.
  - Requires the **optimism gap** (internal-CV minus external-cohort) be reported per benchmark, and
    a (code version, data version, split hash) **provenance triple** on every number.
  - Adopts a TRIPOD-AI-aligned reporting template; reviewed and signed off by the domain reviewer.

- **ml-oncology-benchmarks-gate-002** (licence/access/de-id gate)
  - Records per source: `accessTier` (open/aggregate/de-identified — **controlled ⇒ EXCLUDE**),
    `licenceId` + `licenceUrl` + cited clause, `permitsResearchReuse`, `permitsDerivedRedistribution`,
    `deidentification: publisher-documented`, attribution string, and a licence **snapshot**
    (committed copy + SHA-256 + Wayback URL).
  - Encodes the source classification: TCGA/GDC **open tier**, GEO, CPTAC open, PRIDE (CC-BY/CC0)
    = acceptable; **controlled GDC tiers, dbGaP, EGA, individual-level SEER/biobanks = EXCLUDED**;
    COSMIC/OncoKB = **restricted (NC)**, usable only under NC terms with inherited restriction.
  - Any missing field, unclear licence, or controlled/identifiable signal = **FLAG/EXCLUDE**, never
    default-allow; licence+de-id reviewer sign-off recorded.

- **ml-oncology-benchmarks-audit-004** (leakage-audit skeleton)
  - Implements at minimum the **patient-overlap** check (no case ID across splits, incl. internal vs
    external cohort) and the **preprocessing-isolation** check (static + runtime assertion that no
    test data is seen during any `fit`).
  - Ships **golden fixtures**: known-clean cases that must pass and deliberately-leaky cases that
    must fail; the suite runs in CI and **fails the build** on any violation.
  - Any leak discovered later is added as a non-decreasing regression case within one release.

- **ml-oncology-benchmarks-datasheet-005** (first gated dataset + datasheet)
  - Dataset passes `gate-002` with a committed gate artifact (open/aggregate/de-id + verified
    licence); controlled/identifiable data is rejected.
  - Datasheet (Datasheets-for-Datasets) records motivation, composition, collection, publisher
    de-identification, licence, **batch/centre structure**, and known biases.
  - No row-level/identifiable data is committed; only metadata + provenance.

**M0 Definition of Done:** methodology spec (domain-reviewed) + licence/access/de-id gate published;
repo + TS/Python + green CI; leakage-audit skeleton passing in CI with golden fixtures; ≥ 1 dataset
gated-in with a published datasheet; ≥ 1 adopter/expert outreach thread opened with a dated decision
rule. No model is trained in M0.

---

## Milestone M1 — Reference subtyping benchmark v1

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ml-oncology-benchmarks-subtype-007 | Subtyping benchmark definition object + patient-grouped splits + declared external cohort | data | medium | medium | dataset | 001, 002, 005 | Domain reviewer + Licence+de-id reviewer |
| ml-oncology-benchmarks-audit-008 | Full leakage-audit suite (add feature/label-contamination, duplicate-sample, batch/centre-confound checks) | code | large | medium | pr | 004, 007 | Technical reviewer + Domain reviewer |
| ml-oncology-benchmarks-baselines-009 | Subtyping baselines (fit-on-train-only) + class-correct metrics + optimism-gap report | code | medium | medium | pr | 007, 008 | Domain reviewer + Technical reviewer |
| ml-oncology-benchmarks-modelcards-010 | Model cards + provenance ledger (code+data+split triple per number) | code | medium | medium | pr | 009 | Technical reviewer + Domain reviewer |
| ml-oncology-benchmarks-repro-011 | One-command reproducibility re-run (pinned env, seeds) reproducing all M1 numbers within tolerance | code | small | medium | pr | 009, 010 | Technical reviewer |

**Acceptance criteria — key tasks**

- **ml-oncology-benchmarks-subtype-007** (subtyping benchmark definition)
  - Subtype **label definition is cited to literature** and confirmed current by the domain reviewer;
    feature set + its provenance recorded in the definition object.
  - Splits are **patient/case-grouped**; an **external validation cohort from a different study**
    (e.g. an independent GEO series) is declared and its independence verified by the domain reviewer;
    split-membership files + content hashes emitted.
  - All sources passed `gate-002`; provenance recorded.

- **ml-oncology-benchmarks-baselines-009** (subtyping baselines)
  - Baselines trained with **all preprocessing fit on training folds only** (audit-enforced).
  - Reports **balanced accuracy, macro-F1, AUROC/AUPRC** with confidence intervals on **both**
    internal CV and the external cohort, and the **optimism gap** between them.
  - No leakage-audit check fails in CI; reported numbers carry the provenance triple.

- **ml-oncology-benchmarks-modelcards-010** (model cards + provenance)
  - Each model card states intended use ("research benchmark, **not** a clinical/diagnostic tool"),
    data, metrics, limitations, and the (code, data, split) triple for every number.
  - A provenance ledger maps every published number to its computation; "no source, no claim"
    enforced by a coverage test.

- **ml-oncology-benchmarks-repro-011** (reproducibility re-run)
  - A single command in the pinned container reproduces every M1 number within the declared
    tolerance; a CI job runs it; drift fails the gate.

**M1 Definition of Done:** one subtyping benchmark fully defined on gated-in open data; full
leakage-audit suite green in CI; baselines reported on internal CV **and** an independent external
cohort with the optimism gap quantified; model cards with provenance triples; independent re-run
reproduces numbers within tolerance; domain + licence/de-id + technical sign-offs recorded.

---

## Milestone M2 — Survival benchmark + task-agnostic harness

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ml-oncology-benchmarks-survival-012 | Survival benchmark definition (right-censored endpoint, nested CV, external cohort) | data | medium | medium | dataset | 001, 002, 005 | Domain reviewer + Licence+de-id reviewer |
| ml-oncology-benchmarks-survmetrics-013 | Censoring-aware metric library (Harrell + Uno C-index, time-dependent AUC, integrated Brier score) | code | medium | medium | pr | 001 | Domain reviewer + Technical reviewer |
| ml-oncology-benchmarks-survaudit-014 | Temporal/lookahead leakage check (no post-baseline/outcome-derived predictors) added to audit | code | medium | medium | pr | 008, 012 | Technical reviewer + Domain reviewer |
| ml-oncology-benchmarks-survbaselines-015 | Survival baselines (Cox/penalized/RSF) fit-on-train-only + optimism-gap report | code | medium | medium | pr | 012, 013, 014 | Domain reviewer + Technical reviewer |
| ml-oncology-benchmarks-harness-016 | Refactor harness to run both task families from definition objects (shared audit/metric/provenance code) | code | large | medium | pr | 009, 015 | Technical reviewer + Maintainer |

**Acceptance criteria — key tasks**

- **ml-oncology-benchmarks-survival-012** (survival benchmark definition)
  - Endpoint (OS/PFS) and **censoring definition** cited and confirmed by the domain reviewer;
    patient-grouped, **nested CV**, declared independent external cohort.
  - Sources passed `gate-002`; aggregate-only where individual-level would be controlled.

- **ml-oncology-benchmarks-survmetrics-013** (survival metrics)
  - Implements Harrell **and** Uno C-index, time-dependent AUC, and integrated Brier score with
    correct handling of right-censoring; unit-tested against known reference values.
  - Naive (uncensored) accuracy/AUC paths are rejected by the audit when applied to survival data.

- **ml-oncology-benchmarks-survaudit-014** (temporal leakage check)
  - Flags any predictor derived from post-baseline information or from the outcome itself; CI fails
    on violation; a leaky golden fixture is included.

**M2 Definition of Done:** one survival benchmark with censoring-aware metrics, nested CV, and an
external cohort; temporal-leakage check active in CI; harness refactored so both task families run
from definition objects on shared code; optimism gap reported; M1 sign-offs repeated for survival.

---

## Milestone M3 — Submission protocol, leaderboard & breadth

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ml-oncology-benchmarks-submission-017 | Submission protocol + reproducibility re-run gate (accept only if numbers re-run in-tolerance and data is in-scope/gated) | code | medium | medium | pr | 011, 016 | Technical reviewer + Maintainer |
| ml-oncology-benchmarks-leaderboard-018 | Provenance-gated leaderboard (every row: external-cohort number + code+data+split provenance) | code | medium | medium | pr | 010, 017 | Technical reviewer + Domain reviewer |
| ml-oncology-benchmarks-breadth-019 | Second cancer type/cohort benchmark added via definition object (gate + audit + external cohort) | data | large | medium | dataset | 002, 016 | Domain reviewer + Licence+de-id reviewer |

**Acceptance criteria — key tasks**

- **ml-oncology-benchmarks-submission-017** (submission protocol)
  - A submission is accepted **only if** its data passed the gate (open/aggregate/de-id) **and** its
    reported numbers re-run within tolerance in the pinned environment; otherwise rejected with a
    reason.
  - Submissions must include their (code, data, split) provenance; the leakage audit is run on the
    submission's split/preprocessing plan.

- **ml-oncology-benchmarks-leaderboard-018** (provenance-gated leaderboard)
  - Every leaderboard row shows an **external-cohort** metric (not internal CV) and links to its
    provenance triple; rows without provenance or external validation are not listed.
  - Leaderboard is published CC-BY-4.0; NC-derived entries carry the inherited restriction label.

**M3 Definition of Done:** submission protocol with a working reproducibility re-run gate; a
provenance-gated leaderboard live; ≥ 2 benchmarks across ≥ 2 cancer types/cohorts; ≥ 1 external
submission passes the re-run gate.

---

## Milestone M4 — Adoption, reproduction & sustainability (the deed)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ml-oncology-benchmarks-adopt-020 | Secure adopter/steward + record verified external reproductions and uses | research | medium | medium | document | 011, 016, 018 | Steward + Maintainer + Domain reviewer |
| ml-oncology-benchmarks-ops-021 | Ops/maintenance runbook + outcome dashboard + data/licence re-verification cadence + maintenance rotation | maintenance | medium | medium | document | 020 | Maintainer + Steward |
| ml-oncology-benchmarks-edu-022 | (Optional) Patient-facing "what survival/subtyping models can and cannot tell you" explainer — education only | writing | medium | high | document | 020 | Oncologist + Advocate (blocking) |

**Acceptance criteria — key tasks**

- **ml-oncology-benchmarks-adopt-020** (secure adopter — the deed)
  - A real lab/consortium/reproducibility group adopts the benchmark; steward assigned;
    `verifiedNeed` flips `true`.
  - ≥ 2 **verified external reproductions** (independent re-run matches within tolerance) and ≥ 3
    **verified external uses** (citation / merged PR / passing submission) recorded.
  - If no adopter is secured by the dated decision point, the **build-vs-mothball/pivot rule** is
    applied and recorded (hand to a teaching/reproducibility venue, or mothball to maintenance-only)
    rather than declaring a deed with no beneficiary.

- **ml-oncology-benchmarks-edu-022** (optional patient-facing explainer — HIGH risk)
  - Education only; carries a prominent **"not medical advice"** notice; gives no clinical/treatment
    recommendation and no individual prognosis.
  - **Blocked on oncologist AND patient-advocate sign-off** (both recorded) before it ships; every
    assertion cited to an authoritative source.

**M4 Definition of Done:** project-level **Definition of Shipped** met — a real adopter uses/reproduces
the benchmark, ≥ 2 reproductions + ≥ 3 external uses recorded, maintenance rotation + re-verification
cadence in place; any patient-facing explainer shipped only with oncologist + advocate sign-off (or
explicitly deferred). *(Gated on a secured adopter — TO BE SECURED.)*

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
|---|---|---|---|---|---|---|
| ml-oncology-benchmarks-proteomics-023 | Proteomic subtyping benchmark on open PRIDE/CPTAC data | data | large | medium | dataset | Per-accession CC-BY/CC0 licence verified; external cohort required |
| ml-oncology-benchmarks-multimodal-024 | Multi-omics (expression + clinical) survival benchmark | data | large | medium | dataset | Only after harness stable; leakage risk higher across modalities |
| ml-oncology-benchmarks-imaging-025 | Open whole-slide pathology benchmark (open TCGA/CPTAC images only) | data | large | medium | dataset | Deferred; large licence + de-id surface; only if clean open-licence path |
| ml-oncology-benchmarks-fairness-026 | Subgroup/disparity evaluation across aggregate strata (no individual data) | code | medium | medium | pr | Aggregate only; no re-identification |
| ml-oncology-benchmarks-funded-027 | Funded-lane large-scale benchmark run under a hard budget cap | code | large | medium | pr | lane=funded; requires `fundedBudgetUsd` cap; never uncapped |
| ml-oncology-benchmarks-tutorial-028 | "How to evaluate oncology ML without leaking" teaching tutorial | writing | medium | low | document | Derived from the methodology spec; CC-BY-4.0 |

---

## Example task JSON

Complete, schema-valid Task JSON for the first M0 task (`ml-oncology-benchmarks-methodology-001`):

```json
{
  "id": "ml-oncology-benchmarks-methodology-001",
  "title": "Leakage-aware benchmark methodology specification (splits, preprocessing isolation, metrics, external-validation requirement, reporting template)",
  "project": "ml-oncology-benchmarks",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer", "oncology", "machine-learning", "reproducibility", "data"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "medium",
  "status": "open",
  "context": "ML results in oncology are widely over-optimistic because of data leakage: feature selection or normalization fit on the full dataset before cross-validation, sample-level splits that place multiple specimens from the same patient on both sides of a train/test boundary, batch/centre effects that let a model predict the lab instead of the biology, signatures trained and reported on the same cohort, duplicate samples across 'independent' cohorts, and survival models scored with metrics that ignore censoring. This specification is the cold-start contract every benchmark in the project must implement and be tested against, BEFORE any model is trained. The project uses ONLY open-access / aggregate / de-identified cancer data; controlled-access (dbGaP/EGA/biobanks) and any identifiable patient data are out of scope. No adopter/steward or domain reviewer is yet secured.",
  "objective": "Write the authoritative leakage-aware benchmark methodology specification that all later harness code, audits, and benchmarks implement and are tested against: the patient/case-grouped split policy, the mandatory external-validation-cohort requirement, the preprocessing-isolation rules (fit on training folds only), the task-correct metric definitions (censoring-aware survival metrics; class-imbalance-aware subtyping metrics), the optimism-gap reporting, the (code, data, split) provenance-triple requirement, and the TRIPOD-AI-aligned reporting template.",
  "acceptanceCriteria": [
    "Mandates patient/case-grouped splitting as the default unit (no patient on both sides of any split) and a declared external validation cohort from a different study/centre for any headline number; internal cross-validation alone is never a published claim",
    "Specifies preprocessing isolation: scaling, imputation, feature selection, PCA, normalization, and batch correction are all fit on training folds only, and defines how this is to be statically and at-runtime enforced by the leakage-audit suite",
    "Defines task-correct metrics: survival = Harrell and Uno C-index, time-dependent AUC, integrated Brier score (censoring-aware); subtyping = balanced accuracy, macro-F1, AUROC/AUPRC, and calibration; explicitly rejects naive accuracy/AUC on censored survival data",
    "Requires the optimism gap (internal-CV minus external-cohort performance) be reported for every benchmark, and a (code version, data version, split hash) provenance triple on every reported number",
    "Adopts a TRIPOD-AI-aligned reporting template and states the 'research benchmark, not a clinical/diagnostic tool, not medical advice' framing carried by all artifacts",
    "Defines the data-scope rule restated from the cancer guardrails: open-access / aggregate / de-identified sources only; controlled-access and identifiable data excluded by hard rule",
    "Reviewed and signed off by the oncology data-science / biostatistics domain reviewer (recorded in the reviewers ledger)"
  ],
  "resources": [
    "planning/projects/ml-oncology-benchmarks/PLAN.md",
    "CLAUDE.md",
    "docs/good-deed-definition.md",
    "packages/schema/src/schemas.ts",
    "planning/ROADMAP.md"
  ],
  "output": "A reviewed methodology-specification document (the leakage-aware benchmark charter) defining the split policy, external-validation requirement, preprocessing-isolation rules, task-correct metrics, optimism-gap and provenance-triple reporting, and the reporting template - the contract that ml-oncology-benchmarks-audit-004 (leakage-audit suite) and every benchmark definition implement and are verified against.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```
