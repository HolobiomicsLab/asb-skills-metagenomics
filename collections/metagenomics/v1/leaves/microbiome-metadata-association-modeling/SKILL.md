---
name: microbiome-metadata-association-modeling
description: Use when comparing whether two sequence variant callers yield comparable
  microbiome-biological-variable association performance, or when building supervised
  classification models from a variant count table (e.g., SvTable.txt) merged with
  sample metadata.
when_to_use_negative:
- 'The goal is detecting true low-abundance variants: HashSeq is not sensitive for
  low-abundance sequences and DADA2 is recommended there (association-parity modeling
  does not solve sensitivity limits).'
- Input is raw FASTQ rather than a variant count table — variant calling (e.g., inferTrueSequences)
  must be run first before metadata association modeling.
- Studying indel variation relative to a parent sequence — HashSeq does not model
  indels; Deblur or alignment-based methods are suggested instead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: HashSeq
  role: Source of one variant count table (SvTable.txt) whose variants are used to
    train the random forest classifiers
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Reference variant caller whose variant table serves as the benchmark for comparing
    HashSeq association performance
- name: Deblur
  role: Additional variant caller whose count table is included in the cross-caller
    performance comparison
- name: Unoise2
  role: Additional variant caller whose count table is included in the cross-caller
    performance comparison
- name: scikit-learn
  role: Trains random forest classifiers (100 trees) and provides RepeatedKFold cross-validation
    for performance evaluation
- name: R
  role: Runs the HashSeq pipeline to generate the variant count tables fed into classification
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:961a521c720903ccb54f9af87945a796376234e8166bec8a7e8dcd0e9e40dd31
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# microbiome-metadata-association-modeling

## Summary

Train random forest classifiers (100 trees, scikit-learn) on 16S rRNA sequence variant count tables (e.g., from HashSeq and DADA2) merged with sample metadata, evaluating association performance via RepeatedKFold cross-validation (4 splits x 10 repeats). Used in the HashSeq paper to show HashSeq variants behave almost identically to DADA2 in supervised classification of gut microbiota data.

## When to use

Use when comparing whether two sequence variant callers yield comparable microbiome-biological-variable association performance, or when building supervised classification models from a variant count table (e.g., SvTable.txt) merged with sample metadata.

## When NOT to use

- The goal is detecting true low-abundance variants: HashSeq is not sensitive for low-abundance sequences and DADA2 is recommended there (association-parity modeling does not solve sensitivity limits).
- Input is raw FASTQ rather than a variant count table — variant calling (e.g., inferTrueSequences) must be run first before metadata association modeling.
- Studying indel variation relative to a parent sequence — HashSeq does not model indels; Deblur or alignment-based methods are suggested instead.

## Inputs

- Sequence variant count tables from HashSeq, DADA2, Deblur, and Unoise2 (e.g., HashSeq SvTable.txt)
- Sample metadata table with biological variables of interest (e.g., autism gut data set PRJNA533120, n = 81)

## Outputs

- Per-caller, per-variable cross-validated random forest performance table (rf_cv_performance.csv)
- Cross-caller comparison figure (e.g., fig7_rf_performance.png)

## How to apply

Load each caller's variant count table (HashSeq, DADA2, Deblur, Unoise2) for the data set of interest (e.g., the autism gut data set PRJNA533120, n = 81) and merge it with the sample metadata variables of interest. Filter variants to those passing the pipeline abundance threshold — total reads >= 1,000 across all samples (10,000 for the largest data set), consistent with HashSeq calling parameters. For each caller's table and each metadata variable, train a random forest classifier with 100 trees in scikit-learn. Evaluate with RepeatedKFold cross-validation (4 splits x 10 repeats) and record mean cross-validated association performance (e.g., AUC) per caller and per variable, then compare across callers. In the source study this was used to test that HashSeq-derived variants yield performance nearly identical to DADA2 (Fig. 7).

## Related tools

- **HashSeq** (Source of one variant count table (SvTable.txt) whose variants are used to train the random forest classifiers) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Reference variant caller whose variant table serves as the benchmark for comparing HashSeq association performance)
- **Deblur** (Additional variant caller whose count table is included in the cross-caller performance comparison)
- **Unoise2** (Additional variant caller whose count table is included in the cross-caller performance comparison)
- **scikit-learn** (Trains random forest classifiers (100 trees) and provides RepeatedKFold cross-validation for performance evaluation)
- **R** (Runs the HashSeq pipeline to generate the variant count tables fed into classification) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import RepeatedKFold, cross_val_score; rf = RandomForestClassifier(n_estimators=100); scores = cross_val_score(rf, X, y, cv=RepeatedKFold(n_splits=4, n_repeats=10))
```

## Evaluation signals

- rf_cv_performance.csv contains exactly one performance row per caller-variable combination.
- HashSeq vs. DADA2 cross-validated performance values are near-identical, matching the near-equal association performance reported in Fig. 7.
- Variant tables are filtered to total reads >= 1,000 across all samples (10,000 for the largest data set) before modeling.
- Each classifier uses exactly 100 trees and is evaluated with RepeatedKFold at 4 splits x 10 repeats.
- Performance metrics (e.g., AUC) are computed per metadata variable, not pooled across variables.

## Limitations

- HashSeq calls a lower number of sequence variants than DADA2 and is conservative; it is not sensitive for detecting true low-abundance sequences, so parity in classification does not imply equal sensitivity.
- Association parity is demonstrated on specific data sets (e.g., gut microbiota, PRJNA533120, n = 81) and may not generalize to all data sets or variable types.
- Compositionality and its artifacts are not addressed by the underlying variant calling, which could affect association results.

## Evidence

- [results] Random forest classification on gut microbiota data sets shows that HashSeq performs nearly identically to DADA2 in association studies between the gut microbiota and biological variables (Fig. 7).: [evidence span withheld — anchor status: not_found]
- [intro] Our approach provides a conservative set of variant calls that is well supported by a reference database and behaves almost identically to DADA2 calls in supervised classification.: "Our approach provides a conservative set of variant calls that is well supported by a reference database and behaves almost identically to DADA2 calls in supervised classification."
- [discussion] This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [methods] Then we run the inferTrueSequences to generate a count table: "Then we run the inferTrueSequences to generate a count table"
- [readme] This count table includes all sequences from step 4. This table can be used for taxonomic assignment and other down-stream analyses.: "This count table includes all sequences from step 4. This table can be
used for taxonomic assignment and other down-stream analyses."
