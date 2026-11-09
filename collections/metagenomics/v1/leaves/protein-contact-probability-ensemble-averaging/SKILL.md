---
name: protein-contact-probability-ensemble-averaging
description: Use when multiple DNN contact-prediction models trained on cross-validation
  splits of the same data (e.g. DeepECA multitask models) are available and the goal
  is to boost per-target contact accuracy by combining their probability outputs.
when_to_use_negative:
- Only a single trained model is available and no cross-validation splits exist to
  average over
- The goal is comparing predictor architectures rather than squeezing accuracy out
  of one model family — meta-predictor comparisons (e.g. vs RaptorX-Contact) need
  different tooling
- Predictions are hard classifications or ranked lists rather than raw probabilities
  — simple averaging requires probability outputs
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: DeepECA
  role: DNN framework whose multitask model produces per-residue-pair contact probabilities
    to be ensemble-averaged
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs used as prediction inputs for each target (set E-value
    0.001, coverage 60%, three iterations)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:f88f2637c5d245f9449af4120e2142e9c4e5765089a3479f886a07904580def5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# protein-contact-probability-ensemble-averaging

## Summary

Averages per-residue-pair contact probabilities from multiple DeepECA models trained on different cross-validation splits (up to 10-fold) to raise contact prediction accuracy over any single model. The number of folds has only a small effect, with the 10-fold ensemble being highest in almost all prediction categories.

## When to use

Use when multiple DNN contact-prediction models trained on cross-validation splits of the same data (e.g. DeepECA multitask models) are available and the goal is to boost per-target contact accuracy by combining their probability outputs.

## When NOT to use

- Only a single trained model is available and no cross-validation splits exist to average over
- The goal is comparing predictor architectures rather than squeezing accuracy out of one model family — meta-predictor comparisons (e.g. vs RaptorX-Contact) need different tooling
- Predictions are hard classifications or ranked lists rather than raw probabilities — simple averaging requires probability outputs

## Inputs

- Pre-trained cross-validation DeepECA models (5-fold and up to 10-fold, from the Google Drive deposit)
- Target protein FASTA files (e.g. CASP11 105 domains, CASP12 55 domains) placed in ./dataset/fasta
- HHblits-generated MSAs for each target
- Ground-truth contact maps (Cβ atoms within 8 Å, Cα for glycine) for accuracy evaluation

## Outputs

- Ensemble-averaged residue-pair contact probability matrix per target (in ./contact_pred)
- Top-L, L/2, L/5, L/10 contact accuracy per range category (Tables 8-10 style results)
- Ensemble vs single-model and fold-count (3/5/7/10) accuracy comparisons

## How to apply

Download the pre-trained cross-validation models from the Google Drive deposit and prepare target FASTA files (e.g. CASP11 with 105 domains, CASP12 with 55 domains) in ./dataset/fasta. Run the DeepECA prediction pipeline (`python pipeline.py`) with the multitask model to produce raw contact probability outputs in ./contact_pred. For each target, average the predicted residue-pair probabilities across the 5 (or 10) cross-validation models to form a single ensemble prediction matrix. Evaluate top-L, L/2, L/5, and L/10 contact accuracy per range category (Medium/Long range) against the true Cβ-Cβ < 8 Å (Cα for glycine) contact definition. Repeat with models trained on 3-, 5-, 7-, and 10-fold splits to compare fold counts; expect the 10-fold ensemble to be highest in almost all categories with small differences among fold counts.

## Related tools

- **DeepECA** (DNN framework whose multitask model produces per-residue-pair contact probabilities to be ensemble-averaged) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs used as prediction inputs for each target (set E-value 0.001, coverage 60%, three iterations))

## Examples

```
python pipeline.py  # then average the raw probability outputs in ./contact_pred across the 5 (or 10) cross-validation models per target
```

## Evaluation signals

- Ensemble accuracy exceeds single-model accuracy in almost all prediction categories on both CASP11 and CASP12
- The single exception is expected: Medium top L/10 on CASP12 may not improve with ensembling
- The 10-fold ensemble yields the highest accuracy in almost all categories, though differences among 3/5/7/10 folds are small
- Per-target output in ./contact_pred is a probability matrix (values in [0,1]) averaged across exactly the number of cross-validation models used
- Evaluation uses top-L, L/2, L/5, and L/10 cutoffs with the Cβ < 8 Å contact definition

## Limitations

- Ensembling does not help every category — Medium top L/10 accuracy on CASP12 was an exception
- Difference among fold counts is small, so the added cost of training more folds (up to 10) yields limited gains
- Training models from scratch takes 2-3 days on an NVIDIA TITAN GTX GPU, so reuse of pre-trained models from the deposit is recommended

## Evidence

- [other] Ensemble averaging raises contact prediction accuracy considerably: "Moreover, we demonstrated that ensemble averaging of our models can raise accuracy."
- [discussion] Our final prediction result for each residue pair was obtained simply by averaging these predicted probabilities: "Our final prediction result for each residue pair was obtained simply by averaging these predicted probabilities"
- [readme] Pre-trained models (for prediction) and datasets (for training) are needed.: "Pre-trained models (for prediction) and datasets (for training) are needed."
- [discussion] we defined a contact if the distance of Cβ atoms of the residue pair was less than 8 Å: "we defined a contact if the distance of Cβ atoms of the residue pair was less than 8 Å"
- [readme] The results will be stored in ./contact_pred and ./ss_pred directory.: "The results will be stored in ./contact_pred and ./ss_pred directory."
