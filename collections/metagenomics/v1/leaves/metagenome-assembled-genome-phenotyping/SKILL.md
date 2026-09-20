---
name: metagenome-assembled-genome-phenotyping
description: Use when characterizing draft MAGs (e.g. biogas reactor bins like unClos_1
  at ~91% or unFirm_1 at ~60% completeness) with present/absent phenotype calls, especially
  to verify a manual reconstruction.
when_to_use_negative:
- Input is already a protein family (Pfam) annotation table — use 'traitar phenotype
  ... from_genes' or skip gene calling instead of rerunning the nucleotide workflow
- The goal is phenotype prediction incorporating other omics data — Traitar only takes
  into account presence/absence of protein families
- A trait from the released software is expected but was never shipped — e.g. the
  plant biomass predictor is not included in the Traitar release
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0621
tools:
- name: Traitar
  role: 'Core phenotype predictor: runs the Prodigal → HMMER Pfam annotation → 67-trait
    classification workflow on MAG assemblies'
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Open reading frame prediction on nucleotide FASTA inputs during the from_nucleotides
    workflow
- name: HMMER (hmmsearch)
  role: Scans predicted coding sequences against Pfam-A.hmm to build protein family
    profiles
- name: pip
  role: Recommended installation channel for traitar from the Python packaging index
- name: GNU parallel
  role: Enables multi-process execution via the -c parameter (e.g. -c 2)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:ce56ce5323614145b68973109ef989abbb5037cb8566bd865a65bf368e0a93df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER (hmmsearch)
  - Prodigal
  - Python
  - Traitar
  - matplotlib
  - pandas
  - pip
  - scipy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# metagenome-assembled-genome-phenotyping

## Summary

Phenotype metagenome-assembled (including incomplete) genomes by running Traitar's phenotype workflow, which predicts 67 bacterial traits from Pfam protein family profiles and compares them against a manual metabolic reconstruction.

## When to use

Use when characterizing draft MAGs (e.g. biogas reactor bins like unClos_1 at ~91% or unFirm_1 at ~60% completeness) with present/absent phenotype calls, especially to verify a manual reconstruction.

## When NOT to use

- Input is already a protein family (Pfam) annotation table — use 'traitar phenotype ... from_genes' or skip gene calling instead of rerunning the nucleotide workflow
- The goal is phenotype prediction incorporating other omics data — Traitar only takes into account presence/absence of protein families
- A trait from the released software is expected but was never shipped — e.g. the plant biomass predictor is not included in the Traitar release

## Inputs

- Nucleotide FASTA assemblies of metagenome-assembled genomes (e.g. biogas reactor bins unClos_1, unFirm_1)
- Sample file with columns: sample_file_name, sample_name, optional category
- Pfam 27.0 (Pfam-A.hmm) HMM models
- Manual metabolic reconstruction table (e.g. Frank et al. Table 4) for comparison

## Outputs

- Gene predictions in <out_dir>/gene_prediction
- Pfam annotation in <out_dir>/pfam_annotation
- Phenotype predictions (67 traits) as heatmaps (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) and flat tables (predictions_majority-vote_combined.txt, predictions_flat_majority-votes_combined.txt)
- Per-phenotype comparison summary against the manual reconstruction (e.g. traitar_phenotype_comparison.csv) with recall and agreement metrics

## How to apply

Install traitar via pip (after python-scipy, python-matplotlib, python-pandas) and fetch Pfam 27.0 HMM models with 'traitar pfam <folder>' or register a local copy with 'traitar pfam --local'. Ensure Prodigal, HMMER hmmsearch, and GNU parallel are on the PATH, then run 'traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>' so Prodigal calls ORFs, hmmsearch assigns Pfam families, and the phenotype models output present/absent calls for the 67 traits. For incomplete genomes, the phypat classifier assigns more phenotypes at the price of more false positives, making it suitable for exploratory analysis, while phypat+PGL improves accuracy. Read per-sample predictions from <out_dir>/phenotype_prediction (e.g. predictions_majority-vote_combined.txt, where 0 = absent, 1 = phypat-only, 2 = phylogeny-aware only, 3 = both classifiers). Tally concordant present and absent calls against the manual reconstruction table and report recall on present phenotypes and agreement on absent phenotypes (targets: 6/7 = 87.5% and 9/11 = 81.8%).

## Related tools

- **Traitar** (Core phenotype predictor: runs the Prodigal → HMMER Pfam annotation → 67-trait classification workflow on MAG assemblies) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Open reading frame prediction on nucleotide FASTA inputs during the from_nucleotides workflow)
- **HMMER (hmmsearch)** (Scans predicted coding sequences against Pfam-A.hmm to build protein family profiles)
- **pip** (Recommended installation channel for traitar from the Python packaging index)
- **GNU parallel** (Enables multi-process execution via the -c parameter (e.g. -c 2))

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides traitar_out -c 2
```

## Evaluation signals

- Recall on reconstruction-inferred present phenotypes reaches 6/7 = 87.5% for unClos_1/unFirm_1
- Agreement on reconstruction-inferred absent phenotypes reaches 9/11 = 81.8%
- Output folder contains gene_prediction, pfam_annotation, and phenotype_prediction subdirectories with predictions_majority-vote_combined.txt where predictions encode 0/1/2/3
- Heatmap files (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) are generated covering all samples in the sample file
- All 67 phenotypes from traits.tsv are scored present/absent per genome

## Limitations

- For incomplete genomes, an absent phenotype prediction may simply be due to the absence of the relevant protein families from the input genome (e.g. unFirm_1 at 60% completeness)
- Genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families; other omics data are not integrated and are only proposed as future work
- The plant biomass predictor is not included in the Traitar release
- Docker usage has a known problem with parallel execution (-c option not guaranteed to work) and output files are owned by root
- Feature-track GFF generation with the from_genes option is experimental and gene-id consistency across gene_gff origins (img, RefSeq, Prodigal) is not guaranteed

## Evidence

- [other] Traitar's phenotype predictions for unClos_1 and unFirm_1 agreed to a great extent with the manual reconstruction by Frank et al., recalling 87.5% (6/7) of the reconstruction-inferred phenotypes and agreeing on 81.8% (9/11) of the absent phenotypes.: [evidence span withheld — anchor status: not_found]
- [discussion] One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes.: "One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes."
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer and finally predict phenotypes: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs"
- [readme] A negative prediction is encoded as 0, a prediction made only by the pure phyletic classifier as 1, one made by the phylogeny-aware classifier by 2 and a prediction supported by both algorithms as 3.: [evidence span withheld — anchor status: external_doc]
