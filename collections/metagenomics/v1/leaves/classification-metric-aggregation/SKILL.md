---
name: classification-metric-aggregation
description: Use when comparing multiple classifiers (e.g., PM-CNN, RF) on multi-class
  microbiome datasets (3–5 status labels) and you need to verify a claim that one
  model has more balanced per-class performance via per-status F1 dispersion.
when_to_use_negative:
- The task is overall (global) model comparison using Kappa or accuracy rather than
  per-status balanced-performance assessment — overall metrics miss class-level dispersion.
- Binary classification results with only one F1 score per model — variance across
  per-status F1 scores is undefined/uninformative with a single class.
- Raw OTU abundance tables without trained-model classification results — this skill
  aggregates reported metric tables, it does not train models.
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: numpy
  role: Compute per-model mean, standard deviation, and variance of per-status F1
    scores (numpy.mean, numpy.std, numpy.var) and rank models by variance
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: matplotlib
  role: Visualize per-status F1 statistics across PM-CNN and baseline models
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: PM-CNN
  role: The phylogeny-based multi-path CNN whose balanced-performance claim (lowest
    per-status F1 variance vs. RF and other baselines) is being verified
  repo: https://github.com/qdu-bioinfo/PM_CNN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:cee6807bcbf43fdb1a7579162866b80d877cf703efbba1440ab4951760fe3bb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# classification-metric-aggregation

## Summary

Aggregates per-status (per-class) F1 scores from multi-class microbiome classification results into mean, standard deviation, and variance statistics per model, then ranks models by F1 variance to test claims of balanced performance (e.g., PM-CNN vs. RF baselines).

## When to use

Use when comparing multiple classifiers (e.g., PM-CNN, RF) on multi-class microbiome datasets (3–5 status labels) and you need to verify a claim that one model has more balanced per-class performance via per-status F1 dispersion.

## When NOT to use

- The task is overall (global) model comparison using Kappa or accuracy rather than per-status balanced-performance assessment — overall metrics miss class-level dispersion.
- Binary classification results with only one F1 score per model — variance across per-status F1 scores is undefined/uninformative with a single class.
- Raw OTU abundance tables without trained-model classification results — this skill aggregates reported metric tables, it does not train models.

## Inputs

- Per-status (per-class) F1 score tables for each model (Supplementary Tables S2 and S4 of the PM-CNN paper)
- Classification results for PM-CNN and baseline models (e.g., RF) on the human oral microbiome dataset (3 statuses: Control, Periodontitis, Gingivitis) and human gut microbiome dataset (5 statuses: Control, IBD, HIV, EDD, CRC)

## Outputs

- f1_variance_summary.csv with one row per model and columns: mean_f1, std_f1, var_f1, variance_rank, claim checks
- Ranking of models by ascending per-status F1 variance (PM-CNN expected lowest)
- Optional matplotlib visualization of per-status F1 dispersion per model

## How to apply

Load the per-status F1 score results for PM-CNN and all baseline models from the classification results tables (Supplementary Tables S2 and S4, Results sections 3.2–3.3). For each model, compute the mean, standard deviation, and variance of its per-status F1 scores with numpy (numpy.mean, numpy.std, numpy.var). Rank all models by ascending F1 variance and check whether PM-CNN holds the lowest rank. Also track RF's variance ranking across the two datasets — the gut dataset (5 classes, 3113 samples, 5597 OTUs) versus the oral dataset (3 classes, 1587 samples, 1554 OTUs) — to assess whether RF's ranking declines as sample and class counts grow. Write per-model statistics and rankings to f1_variance_summary.csv with one row per model and columns mean_f1, std_f1, var_f1, variance_rank, plus claim-verification verdicts.

## Related tools

- **numpy** (Compute per-model mean, standard deviation, and variance of per-status F1 scores (numpy.mean, numpy.std, numpy.var) and rank models by variance) — https://github.com/qdu-bioinfo/PM_CNN
- **matplotlib** (Visualize per-status F1 statistics across PM-CNN and baseline models) — https://github.com/qdu-bioinfo/PM_CNN
- **PM-CNN** (The phylogeny-based multi-path CNN whose balanced-performance claim (lowest per-status F1 variance vs. RF and other baselines) is being verified) — https://github.com/qdu-bioinfo/PM_CNN

## Examples

```
import numpy as np; stats = {m: (np.mean(f1), np.std(f1), np.var(f1)) for m, f1 in per_status_f1.items()}
```

## Evaluation signals

- All computed std and variance values are non-negative for every model.
- The ascending variance ranking places PM-CNN at rank 1 (lowest per-status F1 variance) among PM-CNN, RF, and other baselines.
- f1_variance_summary.csv contains exactly one row per model with all required columns (mean_f1, std_f1, var_f1, variance_rank) populated.
- RF's variance ranking trend across the two datasets (oral: 3 classes/1587 samples; gut: 5 classes/3113 samples) matches the reported claim about RF's ranking declining with growing sample and class counts.
- Per-model mean_f1 falls within [0, 1] since F1 scores are bounded metrics.

## Limitations

- The verdict depends on the reported per-status F1 tables (Supplementary Tables S2 and S4); if those tables are unavailable or reformatted, the aggregation cannot be reproduced.
- PM-CNN's overall Kappa coefficient only slightly improved over RF — the balanced-performance advantage is about F1 dispersion, not overall accuracy, so conclusions differ by metric.
- Class imbalance in the source datasets (e.g., oral dataset: Control 653, Periodontitis 274, Gingivitis 660) can influence per-status F1 values and thus the variance ranking.

## Evidence

- [results] finding on PM-CNN balanced performance: [evidence span withheld — anchor status: not_found]
- [methods] aggregation method: [evidence span withheld — anchor status: not_found]
- [readme] oral dataset class composition: "Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660), which are marked with 0, 1, and 2 respectively."
- [supplementary] gut dataset class composition: "Contains five label types: Control(1418), IBD(993), HIV(360), EDD(222), and CRC(120), which are marked with 0, 1, 2, 3, and 4 respectively."
