---
layout: page
title: RWE from Longitudinal EHR
description: A reproducible pipeline for disease outcomes, imputation, and calibrated causal analysis
img: /assets/img/project1/thumb.png
importance: 1
category: work
related_publications: true
bib: refs
---

> A practical, end-to-end workflow to turn routine EHR data into **reliable real-world evidence (RWE)** for chronic diseases—illustrated with a Multiple Sclerosis (MS) case study comparing high-efficacy DMTs.

---

## Why this project?

Randomized trials rarely capture long-term effectiveness in diverse, real-world populations. Yet EHRs do—if we can (1) curate data well, (2) recover missing longitudinal outcomes, and (3) correct bias when those outcomes are imperfect.  
This project packages a **three-step pipeline** that does exactly that:

1. **Data curation** of structured + unstructured EHR.
2. **Label-efficient phenotyping** of longitudinal outcomes (e.g., relapse) with semi-/weak-supervision.
3. **Calibrated causal analysis** so treatment effect (ATE) estimates remain unbiased even when outcomes are imputed.

---

## Pipeline at a glance

<div class="row justify-content-center project-fig">
  <div class="col-xl-11 col-lg-12 col-md-13">
    {% include figure.liquid
       loading="eager"
       path="/assets/img/project1/flowchart.png"
       title="Overall pipeline"
       class="img-fluid w-100 rounded z-depth-1" %}
  </div>
</div>

---

## 1) Curate analysis-ready EHR

- **Standardize structured data**: diagnoses (PheCode), procedures (CCS), meds (RxNorm), labs (LOINC).
- **Extract unstructured data**: clinical note concepts (UMLS CUIs) via NLP (e.g., NILE / MetaMap / cTAKES).
- **Knowledge-graph feature selection**: start from tools like **ONCE / KESER / ARCH** to assemble disease-relevant features; augment with clinician-suggested terms to avoid omissions.
- **Cohort identification**: replace brittle ICD-only filters with weakly/semi-supervised phenotyping (e.g., MAP, PheCAP, APHRODITE).

<div class="row justify-content-center project-fig">
  <div class="col-xl-11 col-lg-12 col-md-13">
    {% include figure.liquid
       loading="eager"
       path="/assets/img/project1/once_app.png"
       title="ONCE feature explorer"
       class="img-fluid w-100 rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Knowledge-guided feature selection via ONCE app {% cite xiong2025_kmaps -f refs %}.
</div>

---

## 2) Phenotype longitudinal outcomes (label-efficient)

We impute outcomes on a **patient-period grid** (e.g., 3-month windows), combining:

- a small set of **gold-standard labels** (registry or curated chart review),
- **silver-standard surrogates** available for all (codes, key NLP terms, treatment events),
- and **knowledge-graph embeddings** of features.

**LATTE** (label-efficient incident phenotyping) {% cite wen2024_latte -f refs %} stacks four modules:

- **Concept Re-weighting (CR)** — boosts semantically related features.
- **Patient-Periods Attention Network (PPAN)** — prioritizes informative periods.
- **Bi-GRU sequential model** — captures temporal dependencies.
- **Dual-loss training** — pre-train on silver labels, then semi-supervised fine-tuning with gold labels.

<div class="row justify-content-center project-fig">
  <div class="col-xl-11 col-lg-12 col-md-13">
    {% include figure.liquid
       loading="eager"
       path="/assets/img/project1/latte_workflow.png"
       title="LATTE workflow"
       class="img-fluid w-100 rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  LATTE: label-efficient phenotyping for sparse, longitudinal outcomes.
</div>

---

## 3) Calibrated causal analysis

Using imputed outcomes directly can bias ATE if errors differ by treatment or confounders. We therefore:

1. Fit **propensity score (PS)** and **outcome regression (OR)** models (doubly robust base).
2. **Calibrate** imputed outcomes using the gold-labeled subset, regressing true outcomes on treatment, confounders, and imputed values to reduce residual error correlations {% cite cheng2021_ss_ate -f refs %}.
3. Compute **calibrated ATE**, correcting the doubly robust estimator by the estimated bias over labeled data.

---

## Case study: Multiple Sclerosis (MS)

**Question.** Compare 1-year relapse under **B-cell depletion** vs **natalizumab** (first-line high-efficacy DMTs).  
**Outcome.** 3-month relapse status (binary) imputed per period, then **rolling-aggregated** to 1-year relapse.  
**Highlights.**

- Knowledge-graph-guided features + registry gold labels.
- LATTE outperformed common ML baselines; **AUC ≈ 0.79** (vs. Lasso ~0.73, RF ~0.76, XGBoost ~0.74).
- **Ensembling** LATTE + baselines improved to **AUC ≈ 0.81**.
- Calibrated ATE used gold labels to correct residual bias in imputed outcomes.

---

## Try it / resources

- **Tutorial website & example code**: <https://celehs.github.io/rwe-tutorial/>
- Methods referenced: LATTE (label-efficient phenotyping), weak/semisupervised phenotyping (MAP/PheCAP/etc.), ONCE feature explorer, calibrated ATE (double-robust + outcome calibration).

---

## Implementation notes

- **Data schema**: patient-period table (features, demographics, silver label, gold label, label indicator).
- **Training**: silver-label pretraining → semi-supervised fine-tuning; cross-fitting for ensembles.
- **Causal step**: PS + OR via penalized models or ML; calibrate outcomes on labeled subset; compute bias-corrected ATE.
- **Scale**: for ~5k patients, ~100k periods, ~100 features, LATTE training ≈ hours on CPU (parallelization/batching recommended).
