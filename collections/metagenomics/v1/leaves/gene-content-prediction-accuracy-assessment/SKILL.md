---
name: gene-content-prediction-accuracy-assessment
description: Use when validating phylogeny-based gene-family prediction accuracy on
  genomes with known annotations, e.g. measuring per-genome and per-functional-category
  accuracy of PICRUSt predictions from a KO/COG reference table and a 16S marker-gene
  tree.
when_to_use_negative:
- The input community lacks genomes with known KO/COG annotations, so no held-out
  truth exists against which prediction accuracy can be scored
- The goal is metagenome prediction from an OTU table rather than benchmarking gene-content
  prediction accuracy
- Assessment targets viral or eukaryotic metagenome components, which PICRUSt does
  not infer (only bacterial and archaeal 16S marker gene sequences are included)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3308
tools:
- name: PICRUSt
  role: Runs the leave-one-genome-out gene-content prediction via ancestral state
    reconstruction on the pruned marker gene tree
  repo: http://picrust.github.com
- name: COUNT package
  role: Provides Wagner Parsimony ancestral state reconstruction used in gene-content
    prediction
- name: APE R library
  role: Provides ACE ML, ACE REML and ACE PIC ancestral state reconstruction alternatives
    for the prediction step
- name: tax2tree
  role: Supplies the 16S Greengenes tree version used as the phylogenetic backbone
    for holdout validation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:03adcfff6e6b05c329dbad126894d06bc6e4c0fb7d8bfc07b9f953a4045b1d21
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - APE R library
  - COUNT package
  - PICRUSt
  - tax2tree
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-content-prediction-accuracy-assessment

## Summary

A leave-one-genome-out holdout validation protocol that quantifies how accurately PICRUSt's ancestral-state-reconstruction-based gene-content predictions recover true KO/COG annotations, reported per taxon (archaea vs bacteria) and per KEGG/BRITE functional category.

## When to use

Use when validating phylogeny-based gene-family prediction accuracy on genomes with known annotations, e.g. measuring per-genome and per-functional-category accuracy of PICRUSt predictions from a KO/COG reference table and a 16S marker-gene tree.

## When NOT to use

- The input community lacks genomes with known KO/COG annotations, so no held-out truth exists against which prediction accuracy can be scored
- The goal is metagenome prediction from an OTU table rather than benchmarking gene-content prediction accuracy
- Assessment targets viral or eukaryotic metagenome components, which PICRUSt does not infer (only bacterial and archaeal 16S marker gene sequences are included)

## Inputs

- IMG v3.5 gene-by-genome KO/COG annotation table
- Greengenes 16S reference tree (tax2tree version)
- NCBI Taxonomy phylum/genus labels for each genome
- KEGG and COG annotation databases for mapping KOs to functional categories

## Outputs

- Per-genome holdout prediction accuracy table (genome_holdout_accuracies.csv)
- Per-taxon mean accuracies (archaea vs bacteria) with s.d. and n
- Per-functional-category accuracies via KEGG Modules / BRITE hierarchy mapping

## How to apply

Load the IMG v3.5 gene-by-genome KO/COG annotation table and the Greengenes 16S reference tree (the tax2tree version) into the PICRUSt environment. For each sequenced genome in IMG, exclude that genome from the reference gene-by-genome table to build the leave-one-genome-out test dataset, then re-run PICRUSt gene-content prediction for the held-out genome by applying ancestral state reconstruction on the pruned 16S tree and weighting extant and inferred ancestral genomes exponentially by the reciprocal of phylogenetic distance (using, e.g., Wagner Parsimony from COUNT v11.0502 or ACE ML/REML/PIC from the APE R library v2.8). Compare each genome's predicted KO/COG content against its actual held-out annotations and record per-genome prediction accuracy, aggregating per-taxon means for archaea (n=103) and bacteria (n=2,487). Map predicted and observed KOs through KEGG Modules and the BRITE hierarchy to higher-level functional categories and compute per-category accuracies, expecting near-identical archaeal (mean=0.94 +/- 0.04 s.d.) and bacterial (mean=0.95 +/- 0.05 s.d.) accuracy, best prediction of 'housekeeping' functions such as genetic information processing (mean=0.99 +/- 0.03 s.d.), and slightly lower accuracy for laterally transferred, variable categories such as environmental information processing (mean=0.95 +/- 0.04 s.d.). Write per-genome and per-category accuracy tables (e.g. genome_holdout_accuracies.csv) and verify the aggregated means reproduce the reported values.

## Related tools

- **PICRUSt** (Runs the leave-one-genome-out gene-content prediction via ancestral state reconstruction on the pruned marker gene tree) — http://picrust.github.com
- **COUNT package** (Provides Wagner Parsimony ancestral state reconstruction used in gene-content prediction)
- **APE R library** (Provides ACE ML, ACE REML and ACE PIC ancestral state reconstruction alternatives for the prediction step)
- **tax2tree** (Supplies the 16S Greengenes tree version used as the phylogenetic backbone for holdout validation)

## Evaluation signals

- Per-taxon mean accuracies reproduce reported values: archaea mean=0.94 +/- 0.04 s.d. (n=103) and bacteria mean=0.95 +/- 0.05 s.d. (n=2,487)
- Per-category accuracies reproduce reported values: genetic information processing mean=0.99 +/- 0.03 s.d. and environmental information processing mean=0.95 +/- 0.04 s.d.
- Each held-out genome is genuinely absent from the reference gene-by-genome table used for its own prediction (leave-one-genome-out check)
- Predicted and observed KO/COG content are compared on identical annotation identifiers (KO/COG) and mapped consistently through KEGG Modules and BRITE
- Prediction weighting uses exponential weighting by the reciprocal of phylogenetic distance over extant and inferred ancestral genomes

## Limitations

- Genes subject to gain, loss, or lateral transfer will be poorly predicted, yielding lower accuracy and broad confidence intervals for categories such as environmental information processing
- Accuracy depends on the availability of phylogenetically close reference genomes; NSTI or a similar measure should be used to quantify expected prediction accuracy
- Special cases such as laterally transferred elements and intracellular endosymbionts with reduced genomes violate the gene content-phylogeny correlation the method relies on

## Evidence
