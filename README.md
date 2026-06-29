# ml-oncology-benchmarks

> Machine-learning results in oncology are widely irreproducible and routinely **over-optimistic**. The dominant cause is not weak models but **data leakage**: feature selection or normalization perform  ·  **Risk tier:** med  ·  **Status:** planning

Machine-learning results in oncology are widely irreproducible and routinely **over-optimistic**. The dominant cause is not weak models but **data leakage**: feature selection or normalization performed on the full dataset before cross-validation, sample-level splits that put multiple specimens from the *same patient* on both sides of a train/test boundary, batch/centre effects that let a model "predict" the lab rather than the biology, gene signatures trained and reported on the same cohort, duplicate or overlapping samples across "independent" cohorts, and survival models scored with metrics that ignore censoring. The published "AUC 0.99 subtype classifier" or "C-index 0.85 survival model" frequently collapses on a genuinely external cohort. This costs the field years of wasted follow-on work and erodes trust in computational oncology.

**Definition of shipped:** licence/access/de-id gate with recorded provenance; (2) the leakage-audit suite passes in CI; (3) it reports both internal-CV and **external-cohort** performance with the optimism gap quantified; (4) every number is provenance-traced (code+data+split triple) in a model card; (5) 

This is an **Elyos** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/elyos

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
elyos browse
elyos next --repo Elyos-Projects/ml-oncology-benchmarks --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
