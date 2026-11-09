---
name: classifier-performance-comparison
description: Use when you have pooled cross-validation prediction tables from two
  competing phenotype classifiers and need per-taxon (e.g. NCBI phylum) accuracy comparison
  to quantify where a phylogeny-aware model outperforms phyletic-pattern-only prediction.
when_to_use_negative:
- Predictions are not cross-validation outputs but held-out or new-sample predictions,
  so pooling per taxon would not estimate the same quantity.
- A taxon is covered by fewer than five species, where per-taxon accuracy is too unstable
  to compare.
- The goal is overall (not per-taxon) classifier comparison; use genome-level pooled
  accuracy instead.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: Traitar
  role: Source of the phypat and phypat+PGL cross-validation phenotype predictions
    that are pooled per phylum
  repo: https://github.com/hzi-bifo/traitar
- name: pandas
  role: Loading prediction tables, joining labels to NCBI phylum, groupby accuracy
    computation, and writing phylum_accuracies.csv
- name: scipy
  role: Statistical support for the comparison analysis environment
- name: matplotlib
  role: Visualization of the per-phylum accuracy profile (Figure 4)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:808804c5fde6728bbe2d006e914d9bbfb44ec9cd5e4b4246f47b9963146a37ed
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# classifier-performance-comparison

## Summary

Pools cross-validation phenotype predictions from two classifiers (Traitar's phypat and phylogeny-aware phypat+PGL) per NCBI taxon to compare their accuracy profiles, e.g. per-phylum accuracy as reported in Figure 4 of the Traitar paper.

## When to use

Use when you have pooled cross-validation prediction tables from two competing phenotype classifiers and need per-taxon (e.g. NCBI phylum) accuracy comparison to quantify where a phylogeny-aware model outperforms phyletic-pattern-only prediction.

## When NOT to use

- Predictions are not cross-validation outputs but held-out or new-sample predictions, so pooling per taxon would not estimate the same quantity.
- A taxon is covered by fewer than five species, where per-taxon accuracy is too unstable to compare.
- The goal is overall (not per-taxon) classifier comparison; use genome-level pooled accuracy instead.

## Inputs

- GIDEON nested cross-validation prediction table for the phypat classifier (genome, phenotype, predicted label, true label)
- GIDEON nested cross-validation prediction table for the phypat+PGL classifier
- NCBI taxonomy phylum assignment per genome

## Outputs

- phylum_accuracies.csv with one row per NCBI phylum (>=5 species coverage) and accuracy columns for phypat and phypat+PGL
- per-phylum accuracy profile suitable for reproducing Figure 4

## How to apply

Load the GIDEON nested cross-validation prediction tables for both classifiers (phypat and phypat+PGL) with pandas. Join each genome's predicted and true phenotype label to its NCBI phylum assignment, restricting to phyla covered by at least five species. Group predictions by phylum and compute the fraction of correctly predicted phenotype labels per phylum, separately for each classifier. Assemble a one-row-per-phylum table with columns for phypat and phypat+PGL accuracy and write it to phylum_accuracies.csv; validate against reported values (Actinobacteria 84%/81%, Bacteroidetes >90%/89%, Proteobacteria 89%/90%, Firmicutes 91%/90%, Tenericutes 91%/86%) within rounding tolerance. The rationale is that pooling per taxon reveals whether incorporating ancestral protein family gains and losses via maximum-likelihood inference improves classification, as the paper reports for phypat+PGL over phyletic patterns alone.

## Related tools

- **Traitar** (Source of the phypat and phypat+PGL cross-validation phenotype predictions that are pooled per phylum) — https://github.com/hzi-bifo/traitar
- **pandas** (Loading prediction tables, joining labels to NCBI phylum, groupby accuracy computation, and writing phylum_accuracies.csv)
- **scipy** (Statistical support for the comparison analysis environment)
- **matplotlib** (Visualization of the per-phylum accuracy profile (Figure 4))

## Examples

```
import pandas as pd; df = pd.read_csv('gideon_cv_phypat+PGL.tsv', sep='\t').merge(phyla, on='genome'); acc = df.groupby('phylum')['correct'].mean(); acc.to_csv('phylum_accuracies.csv')
```

## Evaluation signals

- Computed per-phylum accuracies match reported values within rounding tolerance: Actinobacteria 84% (phypat 81%), Bacteroidetes >90% (89%), Proteobacteria 89% (90%), Firmicutes 91% (90%), Tenericutes 91% (86%).
- phylum_accuracies.csv contains exactly one row per phylum with both classifier columns populated.
- Only phyla with at least five species coverage in the NCBI taxonomy are retained.
- phypat+PGL accuracy is overall at least as high as phypat, consistent with the paper's claim that the phylogeny-aware model improves accuracy over phyletic-pattern-only prediction.

## Limitations

- Per-phylum accuracy is only meaningful for phyla with at least five species; rarer phyla are excluded.
- Accuracy comparison relies on GIDEON phenotype labels; taxa or phenotypes outside GIDEON (e.g. Bergey's Manual bacteria) are not covered by these cross-validation estimates.
- For incomplete genomes, absence of a phenotype prediction may reflect missing protein families in the input genome rather than classifier error.

## Evidence

- [results] Per-phylum accuracies for the phypat+PGL (phypat) classifier were high with low variance: 84% (81%) for Actinobacteria: [evidence span withheld — anchor status: not_found]
- [discussion] improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall.: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [intro] The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny: "The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny"
- [methods] sudo apt-get install python-scipy python-matplotlib python-pip python-pandas: "sudo apt-get install python-scipy python-matplotlib python-pip python-pandas"
- [discussion] One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes.: "One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families"
