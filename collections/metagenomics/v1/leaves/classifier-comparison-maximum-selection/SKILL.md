---
name: classifier-comparison-maximum-selection
description: Use when a paper reports aggregate counts of samples/phenotypes exceeding
  a performance threshold across multiple classifiers, and a supplementary table gives
  per-classifier metrics per row that must be reconciled.
when_to_use_negative:
- When the claim attributes a threshold count to a single named classifier rather
  than 'one of the two Traitar classifiers' — the max-across-classifiers reduction
  would inflate the count
- When only overall/averaged accuracy figures (phypat 82.6%, phypat+PGL 85.5%) are
  available and no per-phenotype table exists — per-row maxima cannot be computed
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pandas
  role: Load Table S2 and compute row-wise maxima and threshold counts across the
    phypat and phypat+PGL macroaccuracy columns
- name: Python
  role: Scripting environment for the recomputation and comparison workflow
  repo: https://github.com/hzi-bifo/traitar
- name: Traitar
  role: Source of the two phenotype classifiers (phypat, phypat+PGL) whose per-phenotype
    cross-validation macroaccuracy is being compared and reconciled
  repo: https://github.com/hzi-bifo/traitar
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:83b0c8824acad0cbbb2fb4a1250acb7f8c94ed9eaf44c53ada8b4b76c98f2371
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Python
  - Traitar
  - pandas
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classifier-comparison-maximum-selection

## Summary

A verification skill that reconciles two competing per-sample performance metrics (here, Traitar's phypat and phypat+PGL macroaccuracy for 67 GIDEON phenotypes in Table S2) by taking the per-row maximum before thresholding, so reported counts like 53 phenotypes >80% and 26 >=90% macroaccuracy can be independently recomputed and checked.

## When to use

Use when a paper reports aggregate counts of samples/phenotypes exceeding a performance threshold across multiple classifiers, and a supplementary table gives per-classifier metrics per row that must be reconciled.

## When NOT to use

- When the claim attributes a threshold count to a single named classifier rather than 'one of the two Traitar classifiers' — the max-across-classifiers reduction would inflate the count
- When only overall/averaged accuracy figures (phypat 82.6%, phypat+PGL 85.5%) are available and no per-phenotype table exists — per-row maxima cannot be computed

## Inputs

- Supplementary Table S2: cross-validation macroaccuracy table for the 67 GIDEON phenotypes, one row per phenotype, one column per classifier (phypat, phypat+PGL)
- Reported counts to verify: 53 phenotypes with >80% macroaccuracy, 26 phenotypes with >=90% macroaccuracy
- Output path for verification_report.csv

## Outputs

- verification_report.csv containing recomputed counts, reported counts (53, 26), and per-threshold agreement flags
- Per-phenotype best-classifier macroaccuracy series (maximum of phypat and phypat+PGL)

## How to apply

Load the supplementary cross-validation table (Table S2 of the Traitar paper, macroaccuracy for the 67 GIDEON phenotypes) with pandas, keeping the separate macroaccuracy columns for phypat (82.6% overall) and phypat+PGL (85.5% overall). For each phenotype row, reduce the two classifier values to one by taking the maximum, since the article's claim is 'one of the two Traitar classifiers' achieving the threshold. Apply the two thresholds with strict and non-strict comparison respectively: best macroaccuracy strictly greater than 80% and at least 90%. Compare recomputed counts against the reported 53 and 26, record agreement flags per threshold, and write recomputed vs reported counts to verification_report.csv.

## Related tools

- **pandas** (Load Table S2 and compute row-wise maxima and threshold counts across the phypat and phypat+PGL macroaccuracy columns)
- **Python** (Scripting environment for the recomputation and comparison workflow) — https://github.com/hzi-bifo/traitar
- **Traitar** (Source of the two phenotype classifiers (phypat, phypat+PGL) whose per-phenotype cross-validation macroaccuracy is being compared and reconciled) — https://github.com/hzi-bifo/traitar

## Examples

```
import pandas as pd; df = pd.read_csv('table_s2.tsv', sep='\t'); best = df[['phypat','phypat+PGL']].max(axis=1); print((best>80).sum(), (best>=90).sum())
```

## Evaluation signals

- Recomputed >80% count equals the reported 53; recomputed >=90% count equals the reported 26
- Number of rows in the loaded table equals 67 GIDEON phenotypes
- Each per-phenotype best macroaccuracy is >= the corresponding phypat value (max can only raise the metric)
- verification_report.csv contains both thresholds with explicit match/mismatch agreement flags
- Bound checks: all counts are integers in [0, 67]

## Limitations

- The max-across-classifiers selection means a phenotype may exceed a threshold via phypat+PGL alone; it cannot say which classifier dominated without keeping per-classifier columns
- Only presence/absence of protein families feeds Traitar's inference; the macroaccuracy values reflect that feature space and no other omics data
- For incomplete genomes, phenotype metrics may be confounded — the absence of a prediction may reflect missing protein families in the input genome rather than classifier error

## Evidence

- [discussion] phypat+PGL classifier improves accuracy over phyletic-pattern-only prediction: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [discussion] Cross-validation against GIDEON phenotype data: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples."
- [readme] Per-classifier results are accessible separately in the output tree: [evidence span withheld — anchor status: external_doc]
- [discussion] Traitar assigns 67 phenotypes from protein family contents: "assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes."
