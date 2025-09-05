---
layout: page
title: Teacher–Teacher (LINE) for Clinical Language
description: Aligning clinical notes and concept graphs with a lightweight knowledge-alignment module
img: /assets/img/project2/thumb.png
importance: 2
category: work
related_publications: true
---

> **LINE** (LIghtweight kNowledge alignmEnt) is a pragmatic _teacher–teacher_ framework that lets two existing LLMs **teach each other** across data forms (e.g., clinical notes ↔ concept graphs) — _without_ re-pretraining the teachers.

- **Why:** clinical notes are rich but hard to share; concept lists are private-friendly but lossy.
- **What LINE does:** aligns the two representation spaces so each can act as a **proxy** for the other.
- **How:** a tiny projection module + contrastive alignment + a residual-recovery stage.

---

## 1) Problem & intuition

Clinical information comes in **complementary forms** — e.g., a narrative note and its extracted **UMLS concepts**. We often only have access to one form (privacy, governance).  
**Question:** Can the available form _stand in_ for the missing one _without retraining big LLMs_?

**Answer:** use two off-the-shelf teacher models (general LLM + clinical KG LLM), then **align** their embeddings via a tiny module (**LINE**) so that either form yields a **shared representation**.

---

## 2) The teacher–teacher framework

<div class="row justify-content-sm-center">
  <div class="col-xl-11 col-lg-12 col-md-13">
    {% include figure.liquid path="assets/img/project2/line_framework.png" title="Teacher–teacher (LINE) framework: note embeddings (Teacher 1) and concept-graph embeddings (Teacher 2) are projected into a joint space and aligned via contrastive objectives." class="img-fluid w-100 rounded z-depth-1" %}
  </div>
</div>

**Teachers (frozen):**

- **Teacher 1 (general LLM)** — e.g., BGE or GPT-4 embeddings.
- **Teacher 2 (clinical KG LLM)** — e.g., CODER with UMLS relations.

**LINE module (trainable only):**

- **Projection for text**: a single **fully connected** layer.
- **Projection for concepts**: **multi-head graph attention** over UMLS relation types + projection to the joint dimension.

**Alignment objective:**

- Contrastive **pair alignment** (note ↔ mean of concept embeddings) with **data-dependent weight** for coverage (accounts for missed extractions).
- **Relational contrastive** term to preserve UMLS relations in the concept space.

**Residual recovery (stage 2):**

- After a few epochs, compute residuals between text and averaged concepts, then **pull in closest concepts** from the projected dictionary to refine pairs and **continue a few more epochs**.

---

## 3) What you get at inference

- If you only have **concepts**, project via Teacher-2 → LINE to get a **proxy text embedding**.
- If you only have **notes**, project via Teacher-1 → LINE to get a **proxy concept embedding**.
- Either way you land in the **same joint space** for retrieval, classification, or causal pipelines.

---

## 4) Results (high-level summary)

- **Alignment:** LINE improves mean rank / Top-10 accuracy vs. BGE, GPT-4, and CODER on held-out note–concept pairs.
- **Clinical NLP (i2b2 2006/2014 NER):** LINE-projected BGE **outperforms** strong clinical baselines.
- **Concept relations (UMLS):** higher **AUC** than teachers on most relation classes (e.g., parent/child, causative, method-of).
- **RCC recurrence (3-class):** the **gap** between sentence vs concept-only embeddings **shrinks** with LINE projections, enabling concept-only analytics.

---

## 5) Efficient by design

Only the small LINE module is trained; teachers stay frozen.

<div class="row justify-content-center project-fig">
  <div class="col-xl-11 col-lg-12 col-md-13">
    {% include figure.liquid path="assets/img/project2/computational_time_comparison.png" title="Compute: LINE trains in hours/epoch on a single 48GB GPU; re-pretraining/fine-tuning big LLMs is days." class="img-fluid w-100 rounded z-depth-1" %}
  </div>
</div>

---

## 6) Minimal recipe to reproduce

1. Build **paired data**: (note text, extracted UMLS concept list) per section/note.
2. **Embed** notes (Teacher-1) and concepts (Teacher-2).
3. Train **LINE**:
   - Stage 1: contrastive pair alignment + relational contrastive.
   - Stage 2: residual-based concept enrichment; train a few more epochs.
4. Use LINE projections for **retrieval/NER/classification** or as the language component in larger pipelines.
