---
name: phenotype-classification-accuracy-evaluation
description: Use when you have Traitar cross-validation phenotype predictions with
  true phenotype labels and need per-taxonomic-rank accuracy (e.g. per-phylum) rather
  than only overall or per-phenotype accuracy.
when_to_use_negative:
- Input is raw nucleotide/protein FASTA not yet phenotyped — run `traitar phenotype`
  first; this skill evaluates existing predictions, not raw sequences
- You need overall or per-phenotype (single-trait) accuracy rather than accuracy stratified
  by a taxonomic rank such as phylum
- The taxon of interest is covered by fewer than five species, which falls outside
  the evaluated phylum profile
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: Traitar
  role: Produces the phypat and phypat+PGL phenotype predictions from genome-derived
    Pfam protein family profiles that are evaluated per phylum
  repo: https://github.com/hzi-bifo/traitar
- name: pandas
  role: Loads prediction tables, joins labels to phylum assignments, groups by phylum,
    and assembles the accuracy table
- name: scipy
  role: Python dependency installed alongside the analysis stack for the accuracy
    computation workflow
- name: matplotlib
  role: Used to render the per-phylum accuracy profile (Figure 4) from the accuracy
    table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:8162bec97af9bc64ba8834eb2bae047d5e5f23407427700e4b47abd4146e8bbb
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# phenotype-classification-accuracy-evaluation

## Summary

Evaluates Traitar phenotype classifier accuracy (phypat and phypat+PGL) by pooling nested cross-validation predictions per taxon — e.g. computing accuracy for bacterial phyla covered by at least five species in the NCBI taxonomy — to produce per-taxon accuracy tables such as the Figure 4 profile.

## When to use

Use when you have Traitar cross-validation phenotype predictions with true phenotype labels and need per-taxonomic-rank accuracy (e.g. per-phylum) rather than only overall or per-phenotype accuracy.

## When NOT to use

- Input is raw nucleotide/protein FASTA not yet phenotyped — run `traitar phenotype` first; this skill evaluates existing predictions, not raw sequences
- You need overall or per-phenotype (single-trait) accuracy rather than accuracy stratified by a taxonomic rank such as phylum
- The taxon of interest is covered by fewer than five species, which falls outside the evaluated phylum profile

## Inputs

- GIDEON nested cross-validation prediction tables for the phypat classifier (predicted label, true label per genome)
- GIDEON nested cross-validation prediction tables for the phypat+PGL classifier (predicted label, true label per genome)
- NCBI taxonomy assignment mapping each genome/species to its phylum
- List of phenotypes with ground-truth labels from the GIDEON microbiology section

## Outputs

- phylum_accuracies.csv — one row per bacterial phylum with phypat and phypat+PGL accuracy columns
- Per-phylum accuracy profile comparable to Figure 4 of the Traitar paper

## How to apply

Load the GIDEON nested cross-validation prediction tables for the phypat and phypat+PGL classifiers with pandas. Join each genome's predicted and true phenotype label to its NCBI taxonomy assignment (phylum). Restrict to phyla covered by at least five species, then group predictions by phylum and compute the fraction of correctly predicted phenotype labels per phylum for each classifier separately. Assemble a one-row-per-phylum table with columns for phypat and phypat+PGL accuracy and write it out (e.g. phylum_accuracies.csv). Validate against the reported profile: Actinobacteria 84% (81%), Bacteroidetes >90% (89%), Proteobacteria 89% (90%), Firmicutes 91% (90%), Tenericutes 91% (86%), within rounding tolerance.

## Related tools

- **Traitar** (Produces the phypat and phypat+PGL phenotype predictions from genome-derived Pfam protein family profiles that are evaluated per phylum) — https://github.com/hzi-bifo/traitar
- **pandas** (Loads prediction tables, joins labels to phylum assignments, groups by phylum, and assembles the accuracy table)
- **scipy** (Python dependency installed alongside the analysis stack for the accuracy computation workflow)
- **matplotlib** (Used to render the per-phylum accuracy profile (Figure 4) from the accuracy table)

## Examples

```
import pandas as pd; per_phylum = preds.merge(taxonomy, on='genome').groupby('phylum').apply(lambda g: (g['pred']==g['true']).mean()); per_phylum.to_csv('phylum_accuracies.csv')
```

## Evaluation signals

- Per-phylum accuracies match reported values within rounding tolerance: Actinobacteria 84%/81%, Bacteroidetes >90%/89%, Proteobacteria 89%/90%, Firmicutes 91%/90%, Tenericutes 91%/86% (phypat+PGL/phypat)
- Only phyla with at least five covered species are included in the table
- Output table has exactly one row per phylum and columns for both classifiers; accuracy values fall in [0,1]
- Each genome's predicted label is checked against its true GIDEON phenotype label before aggregation
- Tenericutes shows the largest phypat-to-phypat+PGL gain (86% → 91%), consistent with the PGL phylogeny-aware correction

## Limitations

- Accuracy is computed only for phyla covered by at least five species in the NCBI taxonomy; rare phyla are not characterized
- phypat+PGL depends on maximum-likelihood inference of ancestral protein family gains/losses on the species phylogeny, so accuracy gains assume phylogeny-aware correction helps despite horizontal gene transfer
- Per-phylum pooling masks per-phenotype variation; individual phenotypes may differ from the pooled accuracy

## Evidence

- [discussion] In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples.: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy"
- [discussion] improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall.: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [intro] We rigorously tested the performance of our software in cross-validation experiments, on further test data sets and for different taxonomic ranks.: "We rigorously tested the performance of our software in cross-validation experiments, on further test data sets and for different taxonomic ranks."
- [readme] The expert user might also want to access the individual results for each algorithm in the respective sub folders ``phypat`` and ``phypat+PGL``.: [evidence span withheld — anchor status: external_doc]
