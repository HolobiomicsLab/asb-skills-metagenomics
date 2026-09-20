---
name: phenotype-macroaccuracy-threshold-counting
description: Use when validating a published accuracy claim stated as a count of phenotypes
  above macroaccuracy thresholds (e.g. >80%, >=90%) and you have a per-phenotype,
  per-classifier cross-validation accuracy table.
when_to_use_negative:
- Input is raw phenotype prediction output (e.g. predictions_majority-vote_combined.txt)
  rather than a per-phenotype macroaccuracy summary table — you would first need cross-validation
  accuracies, not predictions
- The claim concerns a single fixed classifier's overall accuracy (82.6% or 85.5%)
  rather than a best-of-two-classifier phenotype count
- The phenotype set is not the 67 GIDEON phenotypes (e.g. Bergey's Manual phenotypes),
  since the 53/26 counts are tied to the GIDEON cross-validation table
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the Table S2 TSV and computes the row-wise maximum across the phypat
    and phypat+PGL macroaccuracy columns plus threshold counts
- name: Traitar
  role: Source of the two phenotype classifiers (phypat, phypat+PGL) whose cross-validation
    macroaccuracies are being counted
  repo: https://github.com/hzi-bifo/traitar
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:87df83c7009d2e5d561604e15f0a83c0b64c30f632e6d8cf26ca760a07dbe53e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Python
  - Traitar
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# phenotype-macroaccuracy-threshold-counting

## Summary

Verifies reported phenotype-level performance claims by recomputing, per phenotype, the best macroaccuracy across Traitar's two classifiers (phypat and phypat+PGL) from the Table S2 GIDEON cross-validation table and counting how many of the 67 phenotypes exceed the >80% and >=90% thresholds (expected: 53 and 26).

## When to use

Use when validating a published accuracy claim stated as a count of phenotypes above macroaccuracy thresholds (e.g. >80%, >=90%) and you have a per-phenotype, per-classifier cross-validation accuracy table.

## When NOT to use

- Input is raw phenotype prediction output (e.g. predictions_majority-vote_combined.txt) rather than a per-phenotype macroaccuracy summary table — you would first need cross-validation accuracies, not predictions
- The claim concerns a single fixed classifier's overall accuracy (82.6% or 85.5%) rather than a best-of-two-classifier phenotype count
- The phenotype set is not the 67 GIDEON phenotypes (e.g. Bergey's Manual phenotypes), since the 53/26 counts are tied to the GIDEON cross-validation table

## Inputs

- Table S2-style cross-validation macroaccuracy table (TSV/CSV) with one row per GIDEON phenotype (67 rows) and macroaccuracy columns for the phypat and phypat+PGL classifiers
- Reported counts to verify: 53 phenotypes >80% macroaccuracy and 26 phenotypes >=90% macroaccuracy

## Outputs

- verification_report.csv with recomputed counts, reported counts, and per-threshold match/mismatch flags
- Per-phenotype best-of-two-classifier macroaccuracy series used for the threshold counts

## How to apply

Load the Table S2 cross-validation macroaccuracy table for the 67 GIDEON phenotypes with pandas, keeping the per-classifier macroaccuracy columns for phypat and phypat+PGL. For each phenotype row, compute the row-wise maximum across the two classifier columns — the claim is about phenotypes reaching a threshold 'with one of the two classifiers', not a single fixed classifier. Count rows with best macroaccuracy strictly >80% and rows with best macroaccuracy >=90%; strictness matters (>80% excludes exactly-80%, >=90% includes exactly-90%). Compare the two recomputed counts to the reported values of 53 and 26 and write recomputed counts, reported counts, and per-threshold agreement flags to verification_report.csv. As a sanity anchor, the classifiers' overall macroaccuracies are 82.6% (phypat) and 85.5% (phypat+PGL), so most rows should sit in the 80–90% band.

## Related tools

- **pandas** (Loads the Table S2 TSV and computes the row-wise maximum across the phypat and phypat+PGL macroaccuracy columns plus threshold counts)
- **Traitar** (Source of the two phenotype classifiers (phypat, phypat+PGL) whose cross-validation macroaccuracies are being counted) — https://github.com/hzi-bifo/traitar

## Examples

```
import pandas as pd; df = pd.read_csv('Table_S2.tsv', sep='\t'); best = df[['phypat', 'phypat+PGL']].max(axis=1); print((best > 80).sum(), (best >= 90).sum())
```

## Evaluation signals

- Recomputed count of phenotypes with best macroaccuracy strictly >80% equals 53
- Recomputed count of phenotypes with best macroaccuracy >=90% equals 26
- The accuracy table contains exactly 67 phenotype rows (one per GIDEON phenotype)
- Best macroaccuracy per row equals the max of exactly the phypat and phypat+PGL columns — no third classifier column is included
- Overall macroaccuracy values for the two classifiers match the reported 82.6% (phypat) and 85.5% (phypat+PGL) as a column-level sanity check

## Limitations

- The 53/26 counts are specific to the GIDEON cross-validation experiment and the two released classifiers; they do not extend to the Bergey's Manual phenotyping or to incomplete genomes/SAGs
- A phenotype passing the threshold 'with one of the two classifiers' does not mean both classifiers achieve it — per-classifier counts will be lower than the best-of-two counts
- Threshold strictness is asymmetric: >80% excludes exactly 80%, while >=90% includes exactly 90%; confusing these changes the counts

## Evidence

- [supplementary] reported counts match: [evidence span withheld — anchor status: not_found]
- [discussion] GIDEON cross-validation accuracy: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples."
- [readme] 67 phenotypes scope: "Traitar is a software for characterizing microbial samples from nucleotide or protein sequences. It can accurately phenotype [67 diverse traits](traits.tsv)."
- [discussion] two classifiers named: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
