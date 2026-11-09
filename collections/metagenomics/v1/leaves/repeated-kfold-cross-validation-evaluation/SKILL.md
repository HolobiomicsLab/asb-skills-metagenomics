---
name: repeated-kfold-cross-validation-evaluation
description: Use when two or more sequence variant calling methods (e.g., HashSeq
  vs. DADA2) must be compared for downstream association performance on the same microbiota
  data set, requiring cross-validated classifier metrics per caller per metadata variable.
when_to_use_negative:
- The goal is detecting true low-abundance sequence variants — HashSeq is conservative
  and not sensitive for low-abundance sequences; DADA2 is recommended instead
- Input data have not been variant-called yet (raw FASTQ) — this skill evaluates already-generated
  variant count tables, not the calling itself
- Only one variant caller's features are of interest and no caller-vs-caller comparison
  is intended
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HashSeq
  role: Produces the variant count table (via inferTrueSequences) whose supervised
    classification performance is compared against DADA2
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Reference variant caller whose variant table serves as the benchmark for classifier
    performance parity
- name: scikit-learn
  role: Provides RandomForestClassifier (100 trees) and RepeatedKFold (4 splits x
    10 repeats) for cross-validated association performance evaluation
- name: R
  role: Environment for running HashSeq and preparing/merging variant count tables
    with metadata before classification
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:aa483fbb9a4809eaf573e49567fdefd034004ec538b69fc770883b0f2db51e0b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Repeated-KFold cross-validation evaluation of random forest classifiers

## Summary

Trains random forest classifiers (100 trees in scikit-learn) on 16S rRNA gene sequence variant count tables from competing variant callers (HashSeq vs. DADA2) and evaluates their association performance against sample metadata variables using RepeatedKFold cross-validation (4 splits x 10 repeats), so caller-derived variant sets can be compared on equal footing.

## When to use

Use when two or more sequence variant calling methods (e.g., HashSeq vs. DADA2) must be compared for downstream association performance on the same microbiota data set, requiring cross-validated classifier metrics per caller per metadata variable.

## When NOT to use

- The goal is detecting true low-abundance sequence variants — HashSeq is conservative and not sensitive for low-abundance sequences; DADA2 is recommended instead
- Input data have not been variant-called yet (raw FASTQ) — this skill evaluates already-generated variant count tables, not the calling itself
- Only one variant caller's features are of interest and no caller-vs-caller comparison is intended

## Inputs

- Sequence variant count tables per caller (HashSeq SvTable.txt, DADA2 variant table) for the autism gut data set PRJNA533120 (n = 81)
- Sample metadata table with the biological variables of interest

## Outputs

- rf_cv_performance.csv: one mean cross-validated performance row per caller-variable combination
- fig7_rf_performance.png: cross-caller comparison figure showing HashSeq vs. DADA2 classifier performance

## How to apply

Load the sequence variant count tables (e.g., SvTable.txt produced by inferTrueSequences for HashSeq and the equivalent DADA2 output) for the data set of interest, such as the autism gut data set PRJNA533120 (n = 81), and merge each table with the sample metadata variables of interest. Filter variants using the HashSeq abundance threshold — sequence variants with total reads less than 1,000 across all samples are removed (10,000 for the largest data set) — so every caller is evaluated on an equivalent filtered feature set. For each caller's table and each metadata variable, train a random forest classifier with 100 trees in scikit-learn. Evaluate each classifier with RepeatedKFold cross-validation (4 splits x 10 repeats) and record the mean cross-validated association performance (e.g., AUC) per caller and per metadata variable. The rationale is that supervised classification behavior, rather than raw variant counts, reveals whether HashSeq's conservative variant calls are biologically equivalent to DADA2's, as the paper reports they 'behaves almost identically to DADA2 calls in supervised classification.'

## Related tools

- **HashSeq** (Produces the variant count table (via inferTrueSequences) whose supervised classification performance is compared against DADA2) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Reference variant caller whose variant table serves as the benchmark for classifier performance parity)
- **scikit-learn** (Provides RandomForestClassifier (100 trees) and RepeatedKFold (4 splits x 10 repeats) for cross-validated association performance evaluation)
- **R** (Environment for running HashSeq and preparing/merging variant count tables with metadata before classification) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import RepeatedKFold, cross_val_score; clf = RandomForestClassifier(n_estimators=100); cv = RepeatedKFold(n_splits=4, n_repeats=10); scores = cross_val_score(clf, X, y, cv=cv)
```

## Evaluation signals

- rf_cv_performance.csv contains exactly one performance row per caller-variable combination
- HashSeq vs. DADA2 cross-validated performance values are near-identical for each metadata variable, matching the near-equal association performance reported in Fig. 7
- Variant tables were filtered consistently across callers (total reads >= 1,000 across all samples; 10,000 for the largest data set) before training
- Classifier hyperparameters match the protocol: 100 trees per random forest, RepeatedKFold with 4 splits and 10 repeats
- Mean cross-validated metrics (e.g., AUC) are reported per caller and per metadata variable, not as a single pooled score

## Limitations

- HashSeq is conservative and calls a lower number of sequence variants than DADA2, so absolute feature counts differ even when classification parity holds
- HashSeq is not sensitive for detecting true low-abundance sequences; conclusions about caller parity apply to variants passing the abundance threshold
- Classifier performance depends on the metadata variables chosen; parity on one data set (e.g., autism gut, PRJNA533120) does not guarantee parity elsewhere

## Evidence

- [other] train a random forest classifier with 100 trees in scikit-learn: [evidence span withheld — anchor status: not_found]
- [other] Evaluate each classifier with RepeatedKFold cross-validation: [evidence span withheld — anchor status: not_found]
- [intro] conservative set of variant calls behaves almost identically to DADA2 in supervised classification: "Our approach provides a conservative set of variant calls that is well supported by a reference database and behaves almost identically to DADA2 calls in supervised classification."
- [readme] abundance threshold filtering of sequence variants: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [discussion] HashSeq is not sensitive for detecting true low-abundance sequences: "our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2, to detect low-abundance sequence variants."
