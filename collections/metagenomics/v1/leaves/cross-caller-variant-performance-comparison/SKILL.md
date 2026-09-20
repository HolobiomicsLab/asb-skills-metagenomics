---
name: cross-caller-variant-performance-comparison
description: Use when comparing two or more 16S rRNA gene variant callers on the same
  data set and you need to show their variant tables give near-equivalent supervised
  classification/association performance against metadata variables.
when_to_use_negative:
- Detecting true low-abundance sequence variants is the goal — the article notes HashSeq
  is not sensitive for low-abundance sequences and recommends DADA2 instead
- The study involves indel variation relative to a parent sequence or long-read technologies
  such as PacBio, where HashSeq's one-mismatch, no-indel assumptions fail
- Input is raw FASTQ rather than per-caller variant count tables — variant calling
  (e.g., inferTrueSequences) must be run first
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: HashSeq
  role: De novo variant caller whose count table is the primary benchmark; conservative
    caller compared against DADA2 in supervised classification
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Error-model-based variant caller providing the reference variant table for
    performance parity comparison
- name: Deblur
  role: Comparator variant caller using hamming distances against an upper-bound error
    model
- name: Unoise2
  role: Comparator variant caller using preset parameters for filtering low-abundance
    sequencing
- name: scikit-learn
  role: Random forest classifier (100 trees) with RepeatedKFold cross-validation used
    to score association performance per caller
- name: R
  role: Environment for running HashSeq and the manuscript figure scripts
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:3e3221b94ceb438ad464ce5e1d2f4038236ce3ce4b0e5b62e4e79bcc31263e30
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-caller-variant-performance-comparison

## Summary

Benchmarks whether sequence variant callers (HashSeq vs. DADA2, with Deblur and Unoise2 as comparators) yield comparable downstream association performance by training random forest classifiers on each caller's variant count table and comparing cross-validated performance for gut microbiota metadata variables.

## When to use

Use when comparing two or more 16S rRNA gene variant callers on the same data set and you need to show their variant tables give near-equivalent supervised classification/association performance against metadata variables.

## When NOT to use

- Detecting true low-abundance sequence variants is the goal — the article notes HashSeq is not sensitive for low-abundance sequences and recommends DADA2 instead
- The study involves indel variation relative to a parent sequence or long-read technologies such as PacBio, where HashSeq's one-mismatch, no-indel assumptions fail
- Input is raw FASTQ rather than per-caller variant count tables — variant calling (e.g., inferTrueSequences) must be run first

## Inputs

- Variant count tables (SvTable.txt) from HashSeq, DADA2, Deblur, and Unoise2 for the autism gut data set PRJNA533120 (n = 81)
- Sample metadata table with biological variables of interest
- Abundance filter threshold (1,000 total reads default; 10,000 for the largest data set)

## Outputs

- rf_cv_performance.csv with one mean cross-validated performance row per caller-variable combination
- fig7_rf_performance.png showing cross-caller comparison of random forest association performance

## How to apply

Load the sequence variant count tables (SvTable.txt-style outputs) produced by HashSeq, DADA2, Deblur, and Unoise2 for the autism gut data set (PRJNA533120, n = 81) and merge each with the sample metadata variables of interest. Filter variants to those passing the HashSeq abundance threshold (total reads >= 1,000 across all samples; 10,000 for the largest data set), consistent with the pipeline's calling parameters. For each caller's table and each metadata variable, train a random forest classifier with 100 trees in scikit-learn and evaluate with RepeatedKFold cross-validation (4 splits x 10 repeats), recording mean cross-validated association performance (e.g., AUC) per caller-variable combination. The rationale is that HashSeq is more conservative than DADA2 (calls fewer variants), so equivalence of downstream classification performance — not identical variant sets — is the appropriate parity criterion, as reported in Fig. 7 of the manuscript. Write per-caller performance to rf_cv_performance.csv and render the cross-caller comparison as fig7_rf_performance.png.

## Related tools

- **HashSeq** (De novo variant caller whose count table is the primary benchmark; conservative caller compared against DADA2 in supervised classification) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Error-model-based variant caller providing the reference variant table for performance parity comparison)
- **Deblur** (Comparator variant caller using hamming distances against an upper-bound error model)
- **Unoise2** (Comparator variant caller using preset parameters for filtering low-abundance sequencing)
- **scikit-learn** (Random forest classifier (100 trees) with RepeatedKFold cross-validation used to score association performance per caller)
- **R** (Environment for running HashSeq and the manuscript figure scripts) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
rf = RandomForestClassifier(n_estimators=100, random_state=1); cv = RepeatedKFold(n_splits=4, n_repeats=10); scores = cross_val_score(rf, variants, meta, cv=cv)
```

## Evaluation signals

- rf_cv_performance.csv contains exactly one performance row per caller-variable combination
- HashSeq vs. DADA2 performance values are near-identical per metadata variable, matching the near-equal association performance reported in Fig. 7
- Variant tables are filtered consistently to the pipeline threshold (>= 1,000 total reads; 10,000 for the largest data set) before classification
- Each classifier uses 100 trees and RepeatedKFold cross-validation with 4 splits x 10 repeats; mean cross-validated AUC is recorded
- Figure fig7_rf_performance.png renders the cross-caller comparison for all metadata variables

## Limitations

- HashSeq calls a lower number of sequence variants as true than DADA2 and is not sensitive for detecting true low-abundance sequences; parity holds for association performance, not variant-set identity
- HashSeq considers only one-mismatch variants and does not model indels, treating indel events as separate parent sequences
- The default Java heap size (512MB) may be insufficient for large data sets and must be raised via options(java.parameters) before attaching libraries

## Evidence

- [results] Random forest classification on gut microbiota data sets shows that HashSeq performs nearly identically to DADA2 in association studies between the gut microbiota and biological variables (Fig. 7).: [evidence span withheld — anchor status: not_found]
- [intro] behave identically in supervised classification: "Our approach provides a conservative set of variant calls that is well supported by a reference database and behaves almost identically to DADA2 calls in supervised classification."
- [discussion] parent abundance threshold of 1,000 (10,000 largest): "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [discussion] our algorithm, compared to the popular algorithm DADA2, is fairly conservative: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true."
- [readme] sequence variants that have total reads less than 1000 across all samples are removed: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
