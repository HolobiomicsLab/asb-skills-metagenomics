---
name: leave-one-genome-out-holdout-validation
description: Use when benchmarking phylogeny-based functional prediction accuracy
  (e.g., PICRUSt ancestral state reconstruction of KO/COG content) by holding out
  each reference genome in turn and scoring predicted vs. observed annotations per
  taxon and per functional category.
when_to_use_negative:
- When the goal is distance-holdout or tree-randomization validation (excluding all
  genomes within a phylogenetic distance, or randomizing labels on the 16S tree) rather
  than per-genome holdout — those are distinct test designs.
- When evaluating prediction for environments with few related reference genomes,
  where NSTI indicates expected accuracy is too low to benchmark meaningfully.
- When the input is already a 16S OTU table from community samples (e.g., HMP samples)
  rather than the IMG reference genome table — this skill validates genome-level prediction,
  not metagenome inference.
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: PICRUSt
  role: Re-runs gene-content prediction on the pruned 16S tree with the test genome
    excluded from the IMG reference gene-by-genome table
  repo: http://picrust.github.com
- name: APE R library v2.8
  role: Ancestral state reconstruction (ACE ML, ACE REML, ACE PIC) on the pruned marker
    gene tree during re-prediction
- name: COUNT package v11.0502
  role: Wagner Parsimony ancestral state reconstruction alternative used for gene
    gain/loss inference
- name: tax2tree
  role: Provides the tax2tree-annotated Greengenes 16S tree used as the reference
    phylogeny for pruning and reconstruction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:05f00a8d5d0b398af3d828eeadeee024d2830fba653906e0dd9e31a74a39d460
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - APE R library
  - APE R library v2.8
  - COUNT package
  - COUNT package v11.0502
  - PICRUSt
  - tax2tree
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# leave-one-genome-out-holdout-validation

## Summary

A cross-validation scheme that measures how accurately PICRUSt's gene-content prediction recovers a sequenced genome's KO/COG content by excluding that genome from the IMG reference gene-by-genome table and re-predicting it from phylogeny alone, aggregated by taxon and functional category.

## When to use

Use when benchmarking phylogeny-based functional prediction accuracy (e.g., PICRUSt ancestral state reconstruction of KO/COG content) by holding out each reference genome in turn and scoring predicted vs. observed annotations per taxon and per functional category.

## When NOT to use

- When the goal is distance-holdout or tree-randomization validation (excluding all genomes within a phylogenetic distance, or randomizing labels on the 16S tree) rather than per-genome holdout — those are distinct test designs.
- When evaluating prediction for environments with few related reference genomes, where NSTI indicates expected accuracy is too low to benchmark meaningfully.
- When the input is already a 16S OTU table from community samples (e.g., HMP samples) rather than the IMG reference genome table — this skill validates genome-level prediction, not metagenome inference.

## Inputs

- IMG v3.5 gene-by-genome KO/COG annotation table
- Greengenes 16S reference tree (tax2tree version)
- Per-genome taxonomic labels at phylum and genus level from NCBI Taxonomy
- KEGG Modules / BRITE hierarchy mappings for functional-category aggregation

## Outputs

- Per-genome prediction accuracy table for each leave-one-genome-out run
- Per-category accuracy table (e.g., genetic information processing, environmental information processing)
- Per-taxon aggregated means (archaea, bacteria)
- genome_holdout_accuracies.csv

## How to apply

Load the IMG v3.5 gene-by-genome KO/COG annotation table together with the tax2tree-annotated Greengenes 16S reference tree into the PICRUSt environment. For each sequenced genome in IMG, exclude it from the reference gene-by-genome table to build the leave-one-genome-out test set, then re-run PICRUSt gene-content prediction: prune the 16S tree and apply ancestral state reconstruction (Wagner Parsimony from the COUNT package v11.0502 or ACE ML/REML/PIC from the APE R library v2.8), generating predictions as an average of extant and inferred ancestral genome contents weighted exponentially by the reciprocal of phylogenetic distance. Compare each held-out genome's predicted KO/COG content against its actual IMG annotations and record per-genome prediction accuracy. Map predicted and observed KOs through KEGG Modules and the BRITE hierarchy to higher-level functional categories and compute per-category means. Write per-genome and per-category accuracy tables to genome_holdout_accuracies.csv, checking that per-taxon and per-category means reproduce the reported benchmarks.

## Related tools

- **PICRUSt** (Re-runs gene-content prediction on the pruned 16S tree with the test genome excluded from the IMG reference gene-by-genome table) — http://picrust.github.com
- **APE R library v2.8** (Ancestral state reconstruction (ACE ML, ACE REML, ACE PIC) on the pruned marker gene tree during re-prediction)
- **COUNT package v11.0502** (Wagner Parsimony ancestral state reconstruction alternative used for gene gain/loss inference)
- **tax2tree** (Provides the tax2tree-annotated Greengenes 16S tree used as the reference phylogeny for pruning and reconstruction)

## Evaluation signals

- Per-taxon means match reported benchmarks: archaea accuracy = 0.94 +/- 0.04 s.d. (n=103) and bacteria = 0.95 +/- 0.05 s.d. (n=2,487).
- Per-category means match: genetic information processing (housekeeping) = 0.99 +/- 0.03 s.d. and environmental information processing = 0.95 +/- 0.04 s.d.
- For each run, verify the test genome was actually excluded from the reference gene-by-genome table before prediction (prediction uses only pruned-tree extant plus inferred ancestral genomes).
- Output CSV contains both per-genome and per-category accuracy columns, with counts (n) consistent with the number of sequenced genomes in IMG.
- Each genome's predicted KO/COG content is compared against its actual held-out IMG annotations, not against predictions.

## Limitations

- Accuracy depends on availability of related reference genomes; NSTI (nearest sequenced taxon index) must be used to quantify expected prediction accuracy.
- More variable, laterally transferred functional categories (e.g., environmental information processing) are predicted with slightly lower accuracy than core housekeeping functions.
- PICRUSt does not infer viral or eukaryotic metagenome components and cannot distinguish strain-level variation when the marker gene sequence is identical among strains.

## Evidence

- [results] In genome holdout validation, PICRUSt predictions were as accurate for archaeal (mean=0.94 +/- 0.04 s.d., n=103) as for bacterial genomes (mean=0.95 +/- 0.05 s.d., n=2,487): [evidence span withheld — anchor status: not_found]
