---
name: per-status-f1-variance-analysis
description: Use when comparing microbiome classification models on multi-class datasets
  (e.g. 3-class oral or 5-class gut status labels) and the question is whether per-class
  performance is balanced, not just whether overall Kappa or accuracy is higher.
when_to_use_negative:
- The analysis goal is overall performance (e.g. Kappa or accuracy) comparison rather
  than per-class balance — use a standard metric comparison instead.
- Only a single-class or binary task is available with no per-status F1 breakdown,
  so variance across statuses is undefined or trivial.
- Per-class F1 scores for the baseline models are not reported and cannot be recomputed,
  making cross-model variance ranking impossible.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PM-CNN
  role: Model whose per-status F1 variance is being evaluated against baselines; classification
    results come from its training/testing pipeline
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: numpy
  role: Computes per-model mean, standard deviation, and variance of per-status F1
    scores (numpy.mean, numpy.std, numpy.var)
- name: matplotlib
  role: Visualization of per-status F1 dispersion and model rankings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:bd557685cd7d889a5a8d85a093e21037b3c504982c3e2e93f787ec5742280d1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# per-status-f1-variance-analysis

## Summary

Quantifies classification balance by computing mean, standard deviation, and variance of per-status (per-class) F1 scores for PM-CNN versus RF and other baselines, then ranking models by ascending F1 variance. Used to verify the claim that PM-CNN achieves more balanced per-status performance even when overall Kappa only slightly improves over RF.

## When to use

Use when comparing microbiome classification models on multi-class datasets (e.g. 3-class oral or 5-class gut status labels) and the question is whether per-class performance is balanced, not just whether overall Kappa or accuracy is higher.

## When NOT to use

- The analysis goal is overall performance (e.g. Kappa or accuracy) comparison rather than per-class balance — use a standard metric comparison instead.
- Only a single-class or binary task is available with no per-status F1 breakdown, so variance across statuses is undefined or trivial.
- Per-class F1 scores for the baseline models are not reported and cannot be recomputed, making cross-model variance ranking impossible.

## Inputs

- Per-status (per-class) F1 score results for PM-CNN and all baseline models from Supplementary Tables S2 and S4 (Results sections 3.2-3.3)
- Dataset metadata: oral dataset (1587 samples, 1554 OTUs, 3 classes: Control, Periodontitis, Gingivitis) and gut dataset (3113 samples, 5597 OTUs, 5 classes: Control, IBD, HIV, EDD, CRC)

## Outputs

- f1_variance_summary.csv with one row per model and columns: mean_f1, std_f1, var_f1, variance_rank, and claim-check verdicts
- Optional matplotlib figure comparing per-status F1 dispersion across models

## How to apply

Load the per-status F1 scores for PM-CNN and all baseline models (including RF) from the reported classification tables (Supplementary Tables S2 and S4, Results sections 3.2-3.3). For each model, compute mean, standard deviation, and variance of its per-status F1 scores using numpy.mean, numpy.std, and numpy.var. Rank all models by ascending F1 variance and check whether PM-CNN holds the lowest rank; additionally compare RF's ranking across the gut dataset (5 classes, 3113 samples) and oral dataset (3 classes, 1587 samples) to assess whether RF's ranking declines as sample and class counts grow. Write per-model statistics and rankings to f1_variance_summary.csv with one row per model and columns mean_f1, std_f1, var_f1, variance_rank, and claim checks. The claim is verified when PM-CNN has the lowest per-status F1 variance even though its Kappa coefficient only slightly improved over RF.

## Related tools

- **PM-CNN** (Model whose per-status F1 variance is being evaluated against baselines; classification results come from its training/testing pipeline) — https://github.com/qdu-bioinfo/PM_CNN
- **numpy** (Computes per-model mean, standard deviation, and variance of per-status F1 scores (numpy.mean, numpy.std, numpy.var))
- **matplotlib** (Visualization of per-status F1 dispersion and model rankings)

## Examples

```
import numpy as np; mean_f1, std_f1, var_f1 = np.mean(f1_per_status), np.std(f1_per_status), np.var(f1_per_status)
```

## Evaluation signals

- All computed standard deviation and variance values are non-negative.
- PM-CNN holds the lowest variance_rank (rank 1 by ascending per-status F1 variance).
- RF's variance ranking trend across the gut (5-class) and oral (3-class) datasets matches the reported claim that RF's ranking declines as sample and class counts grow.
- f1_variance_summary.csv contains one row per model with all required columns (mean_f1, std_f1, var_f1, variance_rank, claim checks) populated.
- Per-status F1 values parsed from Supplementary Tables S2 and S4 reconcile with the reported Kappa coefficient differences (PM-CNN only slightly above RF).

## Limitations

- Relies on per-status F1 values reported in Supplementary Tables S2 and S4; if those tables or the underlying per-class results are unavailable, the variance ranking cannot be reproduced.
- The oral dataset is class-imbalanced (Periodontitis n=274 vs. Control n=653, Gingivitis n=660), so per-status F1 for minority classes may be unstable.
- The claim is dataset-specific to the two human microbiome datasets (gut and oral); rankings may not generalize to other datasets or class counts.

## Evidence

- [supplementary] finding: per-status variance claim: [evidence span withheld — anchor status: not_found]
- [readme] README: dataset composition: "The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660)"
- [readme] README: gut dataset composition: "The human gut microbiome data contains 3113 samples with 5597 OTUs. Contains five label types: Control(1418), IBD(993), HIV(360), EDD(222), and CRC(120)"
- [readme] README: model training/testing step: "Now, after the processing of Steps 1-3, we now have the clustered feature order. Next, we will start training our model."
