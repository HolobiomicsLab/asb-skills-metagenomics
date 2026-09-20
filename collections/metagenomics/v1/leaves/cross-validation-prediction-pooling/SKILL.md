---
name: cross-validation-prediction-pooling
description: Use when you have nested cross-validation predictions with true phenotype
  labels and need to summarize classifier accuracy per taxon (e.g., bacterial phylum
  with at least five species), as done for Traitar's Figure 4.
when_to_use_negative:
- The goal is an overall (non-taxonomic) accuracy estimate for the classifier — aggregate
  metrics suffice without phylum pooling.
- Taxonomic groups have fewer than five species, making per-group accuracy unstable
  and dominated by individual genomes.
- Predictions come from a single train/test split rather than pooled cross-validation
  runs, so pooling would double-count genomes inconsistently.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Traitar
  role: Source of the phypat and phypat+PGL cross-validation phenotype predictions
    being pooled and scored
  repo: https://github.com/hzi-bifo/traitar
- name: pandas
  role: Loading prediction tables, joining genomes to NCBI phyla, grouping by phylum,
    and writing the accuracy CSV
- name: matplotlib
  role: Optional visualization of per-phylum accuracy profiles analogous to Figure
    4
- name: scipy
  role: Statistical support for accuracy comparisons across taxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:6012aa0e0dcd1c7c68960da82793ccc48ec76e39653b0d2bdde264d4bff3811c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Python
  - Traitar
  - matplotlib
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Cross-validation prediction pooling

## Summary

Pools per-genome cross-validation predictions and true phenotype labels from Traitar classifiers (phypat and phypat+PGL) onto NCBI taxonomic groups (e.g., phyla) to compute per-taxon accuracy profiles. This reveals how classifier accuracy varies across the taxonomy rather than only as an overall mean.

## When to use

Use when you have nested cross-validation predictions with true phenotype labels and need to summarize classifier accuracy per taxon (e.g., bacterial phylum with at least five species), as done for Traitar's Figure 4.

## When NOT to use

- The goal is an overall (non-taxonomic) accuracy estimate for the classifier — aggregate metrics suffice without phylum pooling.
- Taxonomic groups have fewer than five species, making per-group accuracy unstable and dominated by individual genomes.
- Predictions come from a single train/test split rather than pooled cross-validation runs, so pooling would double-count genomes inconsistently.

## Inputs

- GIDEON nested cross-validation prediction tables for the phypat classifier (predicted and true phenotype labels per genome)
- GIDEON nested cross-validation prediction tables for the phypat+PGL classifier (predicted and true phenotype labels per genome)
- NCBI taxonomy assignment (phylum) per genome

## Outputs

- phylum_accuracies.csv: one row per NCBI phylum with columns for phypat accuracy and phypat+PGL accuracy
- Optionally a bar-chart/plot analogous to Figure 4 via matplotlib

## How to apply

Load the GIDEON nested cross-validation prediction tables for the phypat and phypat+PGL classifiers with pandas, one row per genome/phenotype with predicted and true labels. Join each genome's prediction to its NCBI phylum assignment, then group by phylum and compute the fraction of correctly predicted phenotype labels per phylum, keeping the classifiers separate. Restrict to phyla covered by at least five species so the accuracy estimate is not dominated by a handful of genomes. Assemble one row per phylum with columns for phypat and phypat+PGL accuracy and write the table (e.g., phylum_accuracies.csv). Rationale: pooling nested-CV predictions gives a taxon-level view of the high-accuracy, low-variance behavior reported for Traitar, where phypat+PGL reached 84% on Actinobacteria, >90% on Bacteroidetes, 89% on Proteobacteria, 91% on Firmicutes, and 91% on Tenericutes.

## Related tools

- **Traitar** (Source of the phypat and phypat+PGL cross-validation phenotype predictions being pooled and scored) — https://github.com/hzi-bifo/traitar
- **pandas** (Loading prediction tables, joining genomes to NCBI phyla, grouping by phylum, and writing the accuracy CSV)
- **matplotlib** (Optional visualization of per-phylum accuracy profiles analogous to Figure 4)
- **scipy** (Statistical support for accuracy comparisons across taxa)

## Examples

```
import pandas as pd; preds = pd.read_csv('gideoni_predictions_phypat+PGL.tsv', sep='\t'); merged = preds.merge(taxonomy[['genome','phylum']], on='genome'); acc = merged.groupby('phylum').apply(lambda g: (g.prediction == g.true_label).mean()); acc.to_csv('phylum_accuracies.csv')
```

## Evaluation signals

- Computed per-phylum accuracies match the reported values — Actinobacteria 84%/81%, Bacteroidetes >90%/89%, Proteobacteria 89%/90%, Firmicutes 91%/90%, Tenericutes 91%/86% (phypat+PGL/phypat) — within rounding tolerance.
- phylum_accuracies.csv contains exactly one row per NCBI phylum with both classifier columns populated.
- Only phyla with at least five covered species appear in the output table.
- Per-phylum accuracies fall in a plausible 0–1 (or 0–100%) range with the low variance reported for phypat+PGL.

## Limitations

- Accuracy for phyla with few genomes is noisy; the analysis is restricted to phyla covered by at least five species in the NCBI taxonomy.
- Traitar's genotype-phenotype inference uses only presence/absence of protein families; pooling cannot recover accuracy lost to missing signal types.
- For incomplete genomes, an absent phenotype prediction may reflect missing protein families in the input genome rather than classifier error, which can bias pooled accuracy if such genomes are included.

## Evidence

- [results] Per-phylum accuracies for the phypat+PGL (phypat) classifier were high with low variance: 84% (81%) for Actinobacteria, >90% (89%) for Bacteroidetes, 89% (90%) for Proteobacteria, 91% (90%) for Firmicutes, and 91% (86%) for Tenericutes.: [evidence span withheld — anchor status: not_found]
- [discussion] In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples.: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy"
- [intro] We rigorously tested the performance of our software in cross-validation experiments, on further test data sets and for different taxonomic ranks.: "We rigorously tested the performance of our software in cross-validation experiments, on further test data sets and for different taxonomic ranks."
- [readme] The expert user might also want to access the individual results for each algorithm in the respective sub folders ``phypat`` and ``phypat+PGL``.: [evidence span withheld — anchor status: external_doc]
