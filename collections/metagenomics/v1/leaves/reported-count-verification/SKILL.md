---
name: reported-count-verification
description: Use when a paper reports summary statistics (counts of items above thresholds)
  over a per-item results table — such as the Traitar Table S2 macroaccuracy table
  for 67 GIDEON phenotypes — and you must verify the reported numbers against the
  raw data.
when_to_use_negative:
- Input is already an aggregated summary (e.g., overall accuracies like phypat 82.6%
  / phypat+PGL 85.5%) rather than a per-item table — nothing to recount
- The claim concerns prediction outputs on new genomes (use the traitar phenotype
  workflow) rather than verifying counts over existing cross-validation results
- The reported statistic is not a count over per-item rows (e.g., a mean, AUC, or
  effect size) — a different verification approach is needed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3297
tools:
- name: pandas
  role: Loads the Table S2 macroaccuracy table, computes row-wise maxima across classifiers,
    and counts phenotypes above each threshold
- name: Traitar
  role: Source of the two classifiers (phypat, phypat+PGL) whose cross-validation
    macroaccuracy values are being verified
  repo: https://github.com/hzi-bifo/traitar
- name: Python
  role: Execution environment for the verification script
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:2a649811c21e153184b910e60ca670730ea46c2a95b75f9ca4d0b081caef58ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Python
  - Traitar
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3297
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# reported-count-verification

## Summary

A reproducibility skill for recomputing aggregate counts reported in a publication (e.g., '53 of 67 phenotypes >80% macroaccuracy, 26 >=90%') from the underlying per-item data, using pandas to re-derive the numbers and flag agreement or mismatch.

## When to use

Use when a paper reports summary statistics (counts of items above thresholds) over a per-item results table — such as the Traitar Table S2 macroaccuracy table for 67 GIDEON phenotypes — and you must verify the reported numbers against the raw data.

## When NOT to use

- Input is already an aggregated summary (e.g., overall accuracies like phypat 82.6% / phypat+PGL 85.5%) rather than a per-item table — nothing to recount
- The claim concerns prediction outputs on new genomes (use the traitar phenotype workflow) rather than verifying counts over existing cross-validation results
- The reported statistic is not a count over per-item rows (e.g., a mean, AUC, or effect size) — a different verification approach is needed

## Inputs

- Supplementary cross-validation results table (e.g., Table S2) with macroaccuracy per phenotype per classifier (phypat, phypat+PGL) for the 67 GIDEON phenotypes
- Reported aggregate counts from the article text (53 phenotypes >80%; 26 phenotypes >=90%)
- Threshold definitions from the claim (>80% strict, >=90% inclusive)

## Outputs

- verification_report.csv containing recomputed counts, reported counts, and per-threshold agreement flags
- Per-phenotype best (max across classifiers) macroaccuracy series for auditability

## How to apply

Load the per-item results table (Table S2 cross-validation macroaccuracy values for the 67 GIDEON phenotypes, with columns for both classifiers, phypat and phypat+PGL) into pandas, retaining the per-classifier columns. For each phenotype row, compute the maximum macroaccuracy across the two classifiers, since the article claims a phenotype counts if it meets the threshold 'with one of the two Traitar classifiers (phypat: 82.6%; phypat+PGL: 85.5%)'. Count rows whose best value is strictly greater than 80% and rows whose best value is at least 90%, being careful to distinguish strict (>) from inclusive (>=) comparisons, which is the most common source of off-by-one mismatches. Compare the recomputed counts to the reported values (53 >80%, 26 >=90%) and record match/mismatch flags per threshold in a verification_report.csv alongside recomputed and reported values.

## Related tools

- **pandas** (Loads the Table S2 macroaccuracy table, computes row-wise maxima across classifiers, and counts phenotypes above each threshold)
- **Traitar** (Source of the two classifiers (phypat, phypat+PGL) whose cross-validation macroaccuracy values are being verified) — https://github.com/hzi-bifo/traitar
- **Python** (Execution environment for the verification script)

## Examples

```
df = pd.read_csv('table_s2_cv_macroaccuracy.tsv', sep='\t'); best = df[['phypat','phypat+PGL']].max(axis=1); n80=(best>80).sum(); n90=(best>=90).sum(); print(n80, n90)
```

## Evaluation signals

- Recomputed count of phenotypes with best macroaccuracy >80% equals the reported 53
- Recomputed count of phenotypes with best macroaccuracy >=90% equals the reported 26
- The phenotype table contains exactly 67 rows (one per GIDEON phenotype)
- Per-phenotype best values fall within [0, 100] and are consistent with the per-classifier columns (max, not mean or sum)
- verification_report.csv contains explicit match/mismatch flags for both thresholds, all True if verification passes

## Limitations

- Strict vs. inclusive boundary handling matters: a phenotype at exactly 80% counts neither toward >80% nor >=90%, so misreading the claim's threshold semantics changes the count
- The verification depends on the exact supplementary table (Table S2); rounding in the published table versus full-precision values can shift borderline phenotypes across thresholds
- Traitar's genotype-phenotype inference uses only presence/absence of protein families, so the underlying macroaccuracy values reflect that restricted feature space, not multi-omics integration

## Evidence

- [other] 53 phenotypes >80%, 26 >=90% claim: [evidence span withheld — anchor status: not_found]
- [intro] 67 phenotypes framework: "Here we present Traitar, the microbial trait analyzer, an easy-to-use, fully automated software framework for the accurate prediction of currently 67 phenotypes directly from a genome sequence"
- [discussion] classifier improvement: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [readme] combined prediction encoding: [evidence span withheld — anchor status: external_doc]
