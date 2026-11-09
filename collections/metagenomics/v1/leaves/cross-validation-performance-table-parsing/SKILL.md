---
name: cross-validation-performance-table-parsing
description: Use when a paper reports aggregate performance counts over a phenotype/class
  table and you must recompute those counts from a supplementary per-class accuracy
  table to verify the claim.
when_to_use_negative:
- The input is raw genome FASTA or Pfam annotation rather than a precomputed accuracy
  table — run Traitar classification instead of parsing
- The claim involves the independent test data set or Bergey's Manual bacteria rather
  than the 67-phenotype GIDEON cross-validation table
- Only overall mean accuracies (82.6%, 85.5%) are available without per-phenotype
  rows — counts above thresholds cannot be recomputed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: pandas
  role: Loads and manipulates the Table S2 macroaccuracy table and writes verification_report.csv
- name: Traitar
  role: Source of the two classifiers (phypat, phypat+PGL) whose per-phenotype cross-validation
    macroaccuracies are being parsed and verified
  repo: https://github.com/hzi-bifo/traitar
- name: Python
  role: Scripting environment for the counting/verification workflow
  repo: https://github.com/hzi-bifo/traitar
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:45ed3aebfc7f3bb9776f9332f56c189f96836efdd0d41bbc40243e365a4109cc
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-validation-performance-table-parsing

## Summary

Parses a cross-validation macroaccuracy table (Table S2 for the 67 GIDEON phenotypes of Traitar) with pandas, computes per-phenotype best accuracy across two classifiers (phypat and phypat+PGL), and verifies reported count claims (53 phenotypes >80%, 26 >=90%) against recomputed values.

## When to use

Use when a paper reports aggregate performance counts over a phenotype/class table and you must recompute those counts from a supplementary per-class accuracy table to verify the claim.

## When NOT to use

- The input is raw genome FASTA or Pfam annotation rather than a precomputed accuracy table — run Traitar classification instead of parsing
- The claim involves the independent test data set or Bergey's Manual bacteria rather than the 67-phenotype GIDEON cross-validation table
- Only overall mean accuracies (82.6%, 85.5%) are available without per-phenotype rows — counts above thresholds cannot be recomputed

## Inputs

- Table S2 cross-validation macroaccuracy table (67 GIDEON phenotypes) with per-classifier macroaccuracy columns
- Reported counts to verify: 53 phenotypes >80% and 26 phenotypes >=90% macroaccuracy

## Outputs

- verification_report.csv with recomputed counts, reported counts, and per-threshold match/mismatch flags
- Per-phenotype best macroaccuracy values (max across phypat and phypat+PGL)

## How to apply

Load the Table S2 cross-validation macroaccuracy table for the 67 GIDEON phenotypes into a pandas DataFrame, retaining the per-classifier macroaccuracy columns (phypat at 82.6% overall and phypat+PGL at 85.5% overall). For each phenotype row, take the maximum macroaccuracy across the two classifiers, since the claim is achievement 'with one of the two Traitar classifiers'. Count rows whose best accuracy is strictly greater than 80% and rows whose best accuracy is at least 90%, being careful with the strict vs. inclusive threshold semantics. Compare recomputed counts to the reported values of 53 (>80%) and 26 (>=90%) and write agreement flags per threshold to a verification_report.csv. Validation passes only if both recomputed counts equal the reported counts.

## Related tools

- **pandas** (Loads and manipulates the Table S2 macroaccuracy table and writes verification_report.csv)
- **Traitar** (Source of the two classifiers (phypat, phypat+PGL) whose per-phenotype cross-validation macroaccuracies are being parsed and verified) — https://github.com/hzi-bifo/traitar
- **Python** (Scripting environment for the counting/verification workflow) — https://github.com/hzi-bifo/traitar

## Examples

```
df = pd.read_csv('table_S2_macroaccuracy.csv'); best = df[['phypat','phypat+PGL']].max(axis=1); print((best>80).sum(), (best>=90).sum())
```

## Evaluation signals

- Recomputed count of phenotypes with best macroaccuracy strictly >80% equals the reported 53
- Recomputed count of phenotypes with best macroaccuracy >=90% equals the reported 26
- The number of rows parsed equals 67 GIDEON phenotypes with no dropped rows or NaN accuracies
- verification_report.csv contains match/mismatch flags for both thresholds and both flags are True on success

## Limitations

- Verification only checks counts against the reported claims; it cannot detect errors in the underlying Table S2 values themselves
- Threshold semantics matter: the 80% threshold is strict (>80%) while the 90% threshold is inclusive (>=90%); flipping these changes counts
- Phenotype-level absence of prediction can reflect missing protein families in input genomes, so accuracy tables only cover phenotypes evaluated in cross-validation

## Evidence

- [supplementary] counts of 53 and 26: [evidence span withheld — anchor status: not_found]
- [discussion] classifier accuracies: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples."
- [intro] 67 phenotypes: "an easy-to-use, fully automated software framework for the accurate prediction of currently 67 phenotypes directly from a genome sequence"
- [discussion] combined classifiers improve accuracy: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
