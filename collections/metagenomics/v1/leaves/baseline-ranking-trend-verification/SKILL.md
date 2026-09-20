---
name: baseline-ranking-trend-verification
description: Use when a paper claims a model has more balanced per-class performance
  (e.g. lowest per-status F1 variance) and you need to verify that claim by computing
  and ranking F1 variance across models and datasets.
when_to_use_negative:
- Only overall metrics (Kappa, accuracy) are available and per-status F1 scores are
  not reported — variance of per-status F1 cannot be computed.
- The analysis goal is a single-model performance report rather than a cross-model
  balance/ranking comparison against baselines like RF.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: PM-CNN
  role: Model under test whose per-status F1 balance claim (lowest variance across
    Control, Periodontitis, Gingivitis statuses) is verified
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: numpy
  role: Computes per-model mean, standard deviation, and variance of per-status F1
    scores via numpy.mean, numpy.std, numpy.var
- name: matplotlib
  role: Visualizes per-status F1 distributions and variance rankings across models
    and datasets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:af3a789bab53c19346d55bb4fe7dc1012f6c34994b4ba894d843e53d25654d1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# baseline-ranking-trend-verification

## Summary

Computes mean, standard deviation, and variance of per-status (per-class) F1 scores for PM-CNN and baseline models (e.g. RF) from classification result tables, then ranks models by ascending F1 variance to verify claims of balanced classification performance and ranking trends across datasets.

## When to use

Use when a paper claims a model has more balanced per-class performance (e.g. lowest per-status F1 variance) and you need to verify that claim by computing and ranking F1 variance across models and datasets.

## When NOT to use

- Only overall metrics (Kappa, accuracy) are available and per-status F1 scores are not reported — variance of per-status F1 cannot be computed.
- The analysis goal is a single-model performance report rather than a cross-model balance/ranking comparison against baselines like RF.

## Inputs

- Per-status (per-class) F1 score tables for PM-CNN and baseline models (e.g. Supplementary Tables S2 and S4)
- Dataset metadata: oral microbiome dataset (1587 samples, 1554 OTUs, 3 classes: Control/Periodontitis/Gingivitis) and gut microbiome dataset (3113 samples, 5597 OTUs, 5 classes: Control/IBD/HIV/EDD/CRC)

## Outputs

- f1_variance_summary.csv with one row per model and columns mean_f1, std_f1, var_f1, variance_rank, and claim-check verdicts
- Optional matplotlib visualization of per-status F1 variance rankings

## How to apply

Load the per-status (per-class) F1 scores for PM-CNN and all baselines from the reported classification results tables (Supplementary Tables S2 and S4, Results sections 3.2-3.3). For each model compute mean, standard deviation, and variance of its per-status F1 scores with numpy (numpy.mean, numpy.std, numpy.var), then rank all models by ascending variance and check whether PM-CNN holds the lowest rank. Compare RF's ranking across the two datasets (gut: 3113 samples, 5597 OTUs, 5 classes; oral: 1587 samples, 1554 OTUs, 3 classes) to test whether RF's ranking declines as sample and class counts grow. Write per-model statistics, rankings, and claim-verification verdicts to f1_variance_summary.csv with columns mean_f1, std_f1, var_f1, variance_rank, and claim checks.

## Related tools

- **PM-CNN** (Model under test whose per-status F1 balance claim (lowest variance across Control, Periodontitis, Gingivitis statuses) is verified) — https://github.com/qdu-bioinfo/PM_CNN
- **numpy** (Computes per-model mean, standard deviation, and variance of per-status F1 scores via numpy.mean, numpy.std, numpy.var)
- **matplotlib** (Visualizes per-status F1 distributions and variance rankings across models and datasets)

## Examples

```
import numpy as np; mean_f1, std_f1, var_f1 = np.mean(f1), np.std(f1), np.var(f1)  # per model, from Supplementary Tables S2/S4
```

## Evaluation signals

- All computed standard deviation and variance values are non-negative.
- The ascending-variance ranking places PM-CNN at rank 1 (lowest per-status F1 variance) among PM-CNN, RF, and other baselines on the oral microbiome dataset.
- RF's variance rank trend across the gut (5-class) and oral (3-class) datasets matches the reported decline as sample and class counts grow.
- f1_variance_summary.csv contains exactly one row per model with all required columns (mean_f1, std_f1, var_f1, variance_rank, claim checks) populated.
- Computed PM-CNN overall Kappa improvement over RF is only slight, consistent with the reported finding, while per-status variance is lowest for PM-CNN.

## Limitations

- The verification depends on per-status F1 scores being reported in tables such as Supplementary Tables S2 and S4; it cannot be performed from aggregate metrics alone.
- PM-CNN's overall Kappa coefficient only slightly improved over RF, so the balance claim rests on per-status variance rather than a large overall-performance gap.
- The ranking trend claim is assessed on only two datasets with different sample and class counts (gut: 5 classes, oral: 3 classes), limiting generalizability.

## Evidence

- [results] balanced performance finding: [evidence span withheld — anchor status: not_found]
- [full_text] per-status variance workflow: [evidence span withheld — anchor status: not_found]
- [supplementary] supplementary result tables: [evidence span withheld — anchor status: not_found]
- [readme] oral dataset classes: "The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660)"
- [readme] gut dataset classes: "The human gut microbiome data contains 3113 samples with 5597 OTUs. Contains five label types: Control(1418), IBD(993), HIV(360), EDD(222), and CRC(120)"
