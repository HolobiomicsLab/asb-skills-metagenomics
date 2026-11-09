---
name: genotype-phenotype-inference-assessment
description: Use when you need to assess the accuracy of phenotype prediction from
  bacterial genomes, e.g., computing per-phylum accuracies of phypat vs. phypat+PGL
  on pooled GIDEON cross-validation predictions for phyla covered by at least five
  species.
when_to_use_negative:
- Input is raw genome sequences rather than existing cross-validation prediction tables
  — use the Traitar phenotype workflow (traitar phenotype ... from_nucleotides) instead
  of the assessment step
- Phyla (or taxa of interest) are represented by fewer than five species, since the
  per-taxon accuracy profile is defined only for phyla covered by at least five species
- The goal is phenotype prediction itself (e.g., assigning 67 phenotypes to a new
  genome) rather than classifier performance evaluation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3299
tools:
- name: Traitar
  role: Source of the phypat and phypat+PGL classifiers whose cross-validation phenotype
    predictions are evaluated; predicts 67 phenotypes from Pfam protein family profiles
  repo: https://github.com/hzi-bifo/traitar
- name: pandas
  role: Loading prediction tables, joining predictions/labels to phylum assignments,
    grouping by phylum, and writing phylum_accuracies.csv
- name: scipy
  role: Supporting scientific computation for the accuracy profiling analysis
- name: matplotlib
  role: Plotting the per-phylum accuracy profile (Figure 4-style visualization)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:43624b7ddccf7a22f8d6a696d5b7840dda20ddb90f3840a0e18652d450b3b1c8
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# genotype-phenotype-inference-assessment

## Summary

A skill for evaluating genotype-to-phenotype classifiers (e.g., Traitar's phypat and phypat+PGL) by pooling nested cross-validation phenotype predictions per taxon, such as per-phylum accuracy profiles over GIDEON I taxa. It quantifies how well protein-family-based trait prediction generalizes across the NCBI taxonomy.

## When to use

Use when you need to assess the accuracy of phenotype prediction from bacterial genomes, e.g., computing per-phylum accuracies of phypat vs. phypat+PGL on pooled GIDEON cross-validation predictions for phyla covered by at least five species.

## When NOT to use

- Input is raw genome sequences rather than existing cross-validation prediction tables — use the Traitar phenotype workflow (traitar phenotype ... from_nucleotides) instead of the assessment step
- Phyla (or taxa of interest) are represented by fewer than five species, since the per-taxon accuracy profile is defined only for phyla covered by at least five species
- The goal is phenotype prediction itself (e.g., assigning 67 phenotypes to a new genome) rather than classifier performance evaluation

## Inputs

- GIDEON nested cross-validation phenotype prediction tables for the phypat classifier (predictions and true phenotype labels per genome)
- GIDEON nested cross-validation phenotype prediction tables for the phypat+PGL classifier
- NCBI taxonomy phylum assignments for each genome
- Phenotype labels from the GIDEON (Global Infectious Disease and Epidemiology Online Network) microbiology section

## Outputs

- phylum_accuracies.csv: one row per NCBI phylum with columns for phypat and phypat+PGL accuracy
- Per-phylum accuracy profile suitable for reproducing Traitar's Figure 4

## How to apply

Load the GIDEON nested cross-validation prediction tables for the phypat and phypat+PGL classifiers with pandas, then join each genome's predicted and true phenotype label to its NCBI phylum assignment. Restrict to bacterial phyla covered by at least five species, group predictions by phylum, and compute the fraction of correctly predicted phenotype labels per phylum for each classifier separately. Assemble a one-row-per-phylum table with columns for phypat and phypat+PGL accuracy using pandas and write it to phylum_accuracies.csv. The rationale is that phypat+PGL incorporates ancestral protein family gains and losses (maximum-likelihood inference of ancestral character states on the species phylogeny) to account for horizontal gene transfer, so per-taxon profiling reveals where phylogeny-aware inference improves on phyletic-pattern-only prediction.

## Related tools

- **Traitar** (Source of the phypat and phypat+PGL classifiers whose cross-validation phenotype predictions are evaluated; predicts 67 phenotypes from Pfam protein family profiles) — https://github.com/hzi-bifo/traitar
- **pandas** (Loading prediction tables, joining predictions/labels to phylum assignments, grouping by phylum, and writing phylum_accuracies.csv)
- **scipy** (Supporting scientific computation for the accuracy profiling analysis)
- **matplotlib** (Plotting the per-phylum accuracy profile (Figure 4-style visualization))

## Examples

```
import pandas as pd; df = pd.merge(pd.read_csv('gideon_cv_predictions_phypatPGL.csv'), phyla, on='genome'); acc = df.groupby('phylum').correct.mean(); acc.to_csv('phylum_accuracies.csv')
```

## Evaluation signals

- Computed per-phylum accuracies match the reported values within rounding tolerance: phypat+PGL (phypat) Actinobacteria 84% (81%), Bacteroidetes >90% (89%), Proteobacteria 89% (90%), Firmicutes 91% (90%), Tenericutes 91% (86%)
- phylum_accuracies.csv has exactly one row per included phylum and two accuracy columns (phypat, phypat+PGL)
- Only phyla with at least five species coverage in the NCBI taxonomy are retained in the output table
- Accuracies are fractions of correctly predicted phenotype labels computed independently for each classifier from the same pooled cross-validation predictions
- phypat+PGL accuracy should generally be at or above phypat accuracy overall, consistent with the reported improvement over phyletic-pattern-only prediction

## Limitations

- The assessment only takes into account the presence and absence of protein families; other omics data are not integrated
- For incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes, depressing apparent accuracy
- The plant biomass predictor is not included in the released Traitar software, so it cannot be assessed
- Per-phylum estimates are restricted to phyla covered by at least five species in GIDEON, limiting generalization to rare phyla

## Evidence

- [discussion] Cross-validation accuracy profile per phylum: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples."
- [intro] Ancestral state inference rationale: "We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events"
- [intro] Taxonomic-rank testing motivation: "We rigorously tested the performance of our software in cross-validation experiments, on further test data sets and for different taxonomic ranks."
- [readme] Prediction output encoding: [evidence span withheld — anchor status: external_doc]
- [discussion] Genotype-phenotype inference scope: "At the moment, genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families of the bacteria analyzed."
