---
name: hmm-based-genome-annotation
description: Use when starting from nucleotide (or already gene-predicted) bacterial
  genome FASTAs and you need Pfam protein family presence/absence profiles as features
  for trait prediction or comparative genomics, following Traitar's Prodigal + HMMER/Pfam
  27.0 workflow.
when_to_use_negative:
- Protein sequences rather than nucleotide genomes are the input only if you switch
  to Traitar's from_genes/protein path — do not run Prodigal on protein FASTA
- Genes have already been predicted externally with a different caller and GFFs from
  img/RefSeq are used — from_genes feature tracks are experimental and gene IDs are
  inconsistent across sources
- You need trait predictions from other omics data (transcriptomics, metabolomics)
  — Traitar only uses presence/absence of protein families
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0157
tools:
- name: Traitar
  role: 'Orchestrates the annotation pipeline: Prodigal gene prediction, HMMER Pfam
    annotation, and phenotype prediction from the resulting protein family profiles'
  repo: https://github.com/hzi-bifo/traitar
- name: prodigal
  role: Predicts open reading frames from the input nucleotide FASTA genomes before
    Pfam scanning
- name: hmmer/hmmsearch
  role: Scans predicted coding sequences against the Pfam 27.0 HMM models to build
    protein family profiles
- name: GNU parallel
  role: Parallelizes the annotation steps across many genomes via traitar's -c option
- name: pip
  role: Recommended installation method for traitar (pip install traitar-<version>.tar.gz
    --user)
- name: Pfam 27.0 (Pfam-A.hmm)
  role: The specific HMM model set required by Traitar; downloaded with 'traitar pfam'
    or registered with 'traitar pfam --local'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:118d44bdbf643ee81535504c6d3afe644f259572071621f33bbe0e77c03247fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - Pfam 27.0 (Pfam-A.hmm)
  - Python
  - Traitar
  - hmmer/hmmsearch
  - pip
  - prodigal
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hmm-based-genome-annotation

## Summary

Annotate bacterial genomes with protein family profiles by predicting ORFs with Prodigal and scanning the coding sequences against Pfam 27.0 HMM models using HMMER's hmmsearch, as implemented in the Traitar pipeline. These Pfam profiles are the feature basis for downstream genotype-phenotype inference (67 traits) with the phypat and phypat+PGL classifiers.

## When to use

Use when starting from nucleotide (or already gene-predicted) bacterial genome FASTAs and you need Pfam protein family presence/absence profiles as features for trait prediction or comparative genomics, following Traitar's Prodigal + HMMER/Pfam 27.0 workflow.

## When NOT to use

- Protein sequences rather than nucleotide genomes are the input only if you switch to Traitar's from_genes/protein path — do not run Prodigal on protein FASTA
- Genes have already been predicted externally with a different caller and GFFs from img/RefSeq are used — from_genes feature tracks are experimental and gene IDs are inconsistent across sources
- You need trait predictions from other omics data (transcriptomics, metabolomics) — Traitar only uses presence/absence of protein families

## Inputs

- Bacterial genome nucleotide FASTA files (draft or complete) for all samples
- Sample file (tab-separated: sample_file_name, sample_name, optional category column; mandatory header row)
- Pfam 27.0 Pfam-A.hmm HMM models (downloaded via 'traitar pfam' or supplied locally)

## Outputs

- Gene prediction results (Prodigal ORFs) in <out_dir>/gene_prediction
- Pfam protein family annotation (protein family profiles) in <out_dir>/pfam_annotation
- Downstream phenotype predictions for 67 traits in <out_dir>/phenotype prediction (heatmaps and predictions_majority-vote_combined.txt tables)

## How to apply

Install traitar via pip (with prodigal, hmmer's hmmsearch, and GNU parallel on the PATH, verified by 'traitar -h') and obtain the Pfam 27.0 HMM models with 'traitar pfam <download_folder>' or register a local Pfam-A.hmm folder via 'traitar pfam --local'. Build a sample file mapping genome FASTA file names to user sample names (optionally a category column for plotting), then run the phenotype pipeline in 'from_nucleotides' mode so Prodigal predicts open reading frames, hmmsearch annotates the coding sequences with Pfam families, and phenotype models are applied. Use 'from_genes' mode when gene prediction was done externally. Speed up Prodigal/hmmsearch across many genomes (e.g., the 296 Bergey's Manual draft genomes) with the '-c' GNU parallel option. Check that Pfam annotation tables appear in <out_dir>/pfam_annotation before feeding profiles to the phypat / phypat+PGL classifiers.

## Related tools

- **Traitar** (Orchestrates the annotation pipeline: Prodigal gene prediction, HMMER Pfam annotation, and phenotype prediction from the resulting protein family profiles) — https://github.com/hzi-bifo/traitar
- **prodigal** (Predicts open reading frames from the input nucleotide FASTA genomes before Pfam scanning)
- **hmmer/hmmsearch** (Scans predicted coding sequences against the Pfam 27.0 HMM models to build protein family profiles)
- **GNU parallel** (Parallelizes the annotation steps across many genomes via traitar's -c option)
- **pip** (Recommended installation method for traitar (pip install traitar-<version>.tar.gz --user))
- **Pfam 27.0 (Pfam-A.hmm)** (The specific HMM model set required by Traitar; downloaded with 'traitar pfam' or registered with 'traitar pfam --local')

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- 'traitar -h' runs cleanly, confirming prodigal, hmmsearch, and GNU parallel are all reachable on the command line
- The 'traitar pfam' step completes and Pfam 27.0 models are extracted to the specified folder (or accepted via --local)
- <out_dir>/gene_prediction and <out_dir>/pfam_annotation directories are populated for every sample in the sample file
- Downstream phenotype prediction produces the expected artifacts: heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png and predictions_majority-vote_combined.txt with 67 phenotype columns
- On the 296 Bergey's Manual genomes, downstream classification should reach ~72.9% accuracy with phypat and ~72.1% with phypat+PGL, indicating the profiles were built correctly

## Limitations

- For incomplete genomes (e.g., SAGs or draft assemblies), absence of a phenotype prediction may simply reflect missing protein families in the input genome rather than trait absence
- The Docker container has a parallel-usage problem so the -c option is not guaranteed to work, and output files are owned by root
- Pfam 27.0 models are not distributed with traitar and must be downloaded separately
- The from_genes feature-track option with external gene GFFs is experimental; gene IDs are not consistent across img, RefSeq, or Prodigal sources

## Evidence

- [methods] Run traitar's phenotype prediction pipeline, which calls prodigal for gene prediction and hmmsearch against the Pfam 27.0 models: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] Standard workflow predicts ORFs with Prodigal and annotates with Pfam via HMMer: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples"
- [discussion] Annotation is the basis for phenotype prediction from protein family profiles: "We have developed Traitar, a software framework for predicting phenotypes from the protein family profiles of bacterial genomes."
- [readme] Parallel execution across many genomes requires GNU parallel: [evidence span withheld — anchor status: external_doc]
- [methods] Install traitar via pip rather than cloning: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
