---
name: supervised-classifier-training-16s-variants
description: 'Use when comparing variant callers by association performance: variant
  count tables plus sample metadata are available and the goal is to test whether
  different callers yield comparable supervised classification performance.'
when_to_use_negative:
- The goal is to detect true low-abundance sequence variants — HashSeq is conservative
  and not sensitive for low-abundance sequences, so this comparison workflow would
  bias against it
- The task is variant calling itself rather than downstream association analysis —
  use the caller pipelines (HashSeq inferTrueSequences, DADA2) first to generate count
  tables
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: HashSeq
  role: Conservative de novo variant caller whose output count tables (SvTable.txt)
    are one of the tested feature sources for the random forest classifiers
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Reference variant caller; its variants serve as the benchmark against which
    HashSeq classification performance is compared
- name: Deblur
  role: Alternative variant caller whose variant tables are included in the cross-caller
    comparison
- name: Unoise2
  role: Alternative variant caller whose variant tables are included in the cross-caller
    comparison
- name: scikit-learn
  role: Provides the RandomForestClassifier (100 trees) and RepeatedKFold (4 splits
    x 10 repeats) used to train and evaluate classifiers per caller and metadata variable
- name: R
  role: Used to coalesce files and generate manuscript figures from the count tables
    (HashSeq_Manuscript scripts)
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:81d71cdc94dc1cf58d0c25d4145c4259bd9f35cea77adb2a16094ef7321c70e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - Deblur
  - HashSeq
  - R
  - Unoise2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# supervised-classifier-training-16s-variants

## Summary

Train random forest classifiers (100 trees in scikit-learn) on 16S rRNA gene sequence variant count tables from multiple variant callers (HashSeq, DADA2, Deblur, Unoise2) to compare association performance against biological metadata variables, e.g. showing HashSeq performs nearly identically to DADA2 on the gut microbiota autism data set (PRJNA533120, n = 81).

## When to use

Use when comparing variant callers by association performance: variant count tables plus sample metadata are available and the goal is to test whether different callers yield comparable supervised classification performance.

## When NOT to use

- The goal is to detect true low-abundance sequence variants — HashSeq is conservative and not sensitive for low-abundance sequences, so this comparison workflow would bias against it
- The task is variant calling itself rather than downstream association analysis — use the caller pipelines (HashSeq inferTrueSequences, DADA2) first to generate count tables

## Inputs

- Sequence variant count tables (e.g., HashSeq SvTable.txt and equivalent DADA2, Deblur, Unoise2 tables) for the gut microbiota data set PRJNA533120 (n = 81)
- Sample metadata table with biological variables of interest (e.g., autism status)

## Outputs

- rf_cv_performance.csv containing one cross-validated performance row per caller-variable combination
- fig7_rf_performance.png comparing cross-caller association performance

## How to apply

Load the sequence variant count tables (SvTable.txt from HashSeq and equivalent DADA2, Deblur, and Unoise2 tables) produced for the autism gut data set (PRJNA533120, n = 81), and merge each with the sample metadata variables of interest. Filter variants to those passing the HashSeq abundance threshold (total reads >= 1,000 across all samples; 10,000 for the largest data set), matching the caller's filtering parameters. For each caller's table and each metadata variable, train a random forest classifier with 100 trees in scikit-learn and evaluate with RepeatedKFold cross-validation (4 splits x 10 repeats), recording the mean cross-validated performance (e.g., AUC) per caller-variable combination. The rationale is that a conservative caller like HashSeq — which calls fewer variants than DADA2 — should still behave almost identically to DADA2 in supervised classification, as reported in Fig. 7.

## Related tools

- **HashSeq** (Conservative de novo variant caller whose output count tables (SvTable.txt) are one of the tested feature sources for the random forest classifiers) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Reference variant caller; its variants serve as the benchmark against which HashSeq classification performance is compared)
- **Deblur** (Alternative variant caller whose variant tables are included in the cross-caller comparison)
- **Unoise2** (Alternative variant caller whose variant tables are included in the cross-caller comparison)
- **scikit-learn** (Provides the RandomForestClassifier (100 trees) and RepeatedKFold (4 splits x 10 repeats) used to train and evaluate classifiers per caller and metadata variable)
- **R** (Used to coalesce files and generate manuscript figures from the count tables (HashSeq_Manuscript scripts)) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import RepeatedKFold, cross_val_score; scores = cross_val_score(RandomForestClassifier(n_estimators=100), X, y, cv=RepeatedKFold(n_splits=4, n_repeats=10))
```

## Evaluation signals

- rf_cv_performance.csv contains exactly one performance row per caller-variable combination
- HashSeq vs. DADA2 cross-validated performance values are near-identical, matching the near-equal association performance reported in Fig. 7
- Variant tables are filtered to the pipeline abundance threshold (total reads >= 1,000 across all samples; 10,000 for the largest data set) before training
- Each classifier uses 100 trees and is evaluated with RepeatedKFold (4 splits x 10 repeats), so reported metrics are mean cross-validated values (e.g., AUC)
- All four callers (HashSeq, DADA2, Deblur, Unoise2) are evaluated on the same metadata variables for a fair comparison

## Limitations

- HashSeq is not sensitive for detecting true low-abundance sequences; more sensitive algorithms such as DADA2 are recommended for low-abundance variants
- HashSeq is fairly conservative and calls a lower number of sequence variants as true compared to DADA2, which reduces feature richness in the classifier input
- Results are specific to the gut microbiota data set (PRJNA533120, n = 81) and the metadata variables tested; classification parity may not generalize to other data sets

## Evidence

- [results] near-identical classifier performance: [evidence span withheld — anchor status: not_found]
- [intro] behaves almost identically to DADA2: "Our approach provides a conservative set of variant calls that is well supported by a reference database and behaves almost identically to DADA2 calls in supervised classification."
- [discussion] conservative vs DADA2: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true."
- [methods] abundance threshold filter: "sequence variants that have total reads less than 1000 across all samples are removed"
- [discussion] low-abundance sensitivity limitation: "our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2, to detect low-abundance sequence variants."
