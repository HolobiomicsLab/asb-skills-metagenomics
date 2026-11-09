---
name: bacterial-trait-prediction-from-genome-sequence
description: Use when starting from bacterial nucleotide (or protein) FASTA files
  for tens to hundreds of genomes and a genotype-to-phenotype assignment across the
  67 Traitar trait labels is needed without wet-lab phenotyping.
when_to_use_negative:
- Gene/ORF prediction has already been done externally and you have protein or gene
  inputs — use the 'from_genes' mode instead of from_nucleotides
- Your analysis requires integration of transcriptomic or other omics data — Traitar
  only takes into account presence and absence of protein families
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Traitar
  role: 'Orchestrating pipeline: ORF prediction, Pfam annotation, and SVM-based phenotype
    prediction for 67 traits; CLI ''traitar phenotype'''
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from input nucleotide FASTAs in the from_nucleotides
    mode
- name: HMMER
  role: Scans predicted proteins against Pfam 27.0 HMM models to build the protein-family
    profiles used for phenotype prediction
- name: Pfam
  role: Pfam 27.0 HMM models (Pfam-A) used to annotate protein families; not distributed
    with Traitar and must be downloaded
- name: pip
  role: Recommended installation method for traitar
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:e02ffafc0da8a10b161eba8d9076889192a554627cb9f395606375fc65698c9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - HMMER
  - Pfam
  - Prodigal
  - Traitar
  - pip
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bacterial-trait-prediction-from-genome-sequence

## Summary

Predicts 67 bacterial phenotypes directly from a genome nucleotide FASTA by running Traitar's from_nucleotides workflow: Prodigal ORF calling, HMMer/Pfam 27.0 annotation, and pre-trained L1-regularized SVM phenotype models.

## When to use

Use when starting from bacterial nucleotide (or protein) FASTA files for tens to hundreds of genomes and a genotype-to-phenotype assignment across the 67 Traitar trait labels is needed without wet-lab phenotyping.

## When NOT to use

- Gene/ORF prediction has already been done externally and you have protein or gene inputs — use the 'from_genes' mode instead of from_nucleotides
- Your analysis requires integration of transcriptomic or other omics data — Traitar only takes into account presence and absence of protein families

## Inputs

- Nucleotide FASTA file(s) of bacterial genome(s) placed in <in_dir>
- Sample file (tab-separated: sample_file_name, sample_name, optional category column)
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded separately from the package

## Outputs

- phenotype_predictions.tsv (one row per sample, columns covering the 67 traitar phenotype labels)
- Gene prediction results in <out_dir>/gene_prediction
- Pfam annotation in <out_dir>/pfam_annotation
- Phenotype prediction in <out_dir>/phenotype_prediction including heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png, predictions_majority-vote_combined.txt, and predictions_flat_majority-votes_combined.txt
- Feature-track GFF files (phypat/feat_gffs, phypat+PGL/feat_gffs) linking Prodigal-called genes to phenotype-relevant protein families

## How to apply

Run 'traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>', which first predicts open reading frames with Prodigal and translates the coding sequences. Scan the predicted proteins against the Pfam 27.0 HMM models (Pfam-A) with HMMER hmmsearch using the Traitar thresholds of bit-score >= 25 and E value <= 1e-2, then build a binary presence/absence protein-family profile per sample. Apply the pre-trained linear L1-regularized SVM voting committee to each profile to yield present/absent/imprecise assignments for the 67 phenotype labels, via the phypat (phyletic-pattern) and optionally phypat+PGL (phylogeny-aware) classifiers. Traitar requires the Pfam 27.0 HMM models to be downloaded beforehand ('traitar pfam <path>' or registered locally with 'traitar pfam --local'). Parallelism is enabled with -c (requires GNU parallel), and if the output directory exists Traitar offers to recompute or resume individual steps interactively.

## Related tools

- **Traitar** (Orchestrating pipeline: ORF prediction, Pfam annotation, and SVM-based phenotype prediction for 67 traits; CLI 'traitar phenotype') — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from input nucleotide FASTAs in the from_nucleotides mode)
- **HMMER** (Scans predicted proteins against Pfam 27.0 HMM models to build the protein-family profiles used for phenotype prediction)
- **Pfam** (Pfam 27.0 HMM models (Pfam-A) used to annotate protein families; not distributed with Traitar and must be downloaded)
- **pip** (Recommended installation method for traitar)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- phenotype_predictions.tsv exists and contains one row per input sample with columns covering the 67 traitar phenotype labels
- <out_dir>/gene_prediction, <out_dir>/pfam_annotation and <out_dir>/phenotype_prediction directories are all populated
- Prediction table encodes 0 for negative prediction, 1 for phypat-only, 2 for phylogeny-aware-only, and 3 for predictions supported by both classifiers
- Heatmap outputs (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) are generated with hierarchical clustering dendrograms for phenotypes and samples
- Packaged sample data (Listeria grayi DSM_20601 and Listeria ivanovii WSLC3009) completes within ~5 minutes, or the Docker sample run within ~30 minutes

## Limitations

- Plant biomass predictor is not included in the released Traitar software
- For incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes; the phypat classifier assigns more phenotypes to incomplete genomes at the price of more false positives
- Genotype-phenotype inference currently only takes into account presence/absence of protein families; other omics data are not integrated and are only proposed as future work
- Docker output directory is owned by root, so root access is needed to inspect traitar_out, and the -c parallel option is not guaranteed to work in Docker

## Evidence

- [readme] The standard Traitar workflow predicts open reading frames from input nucleotide sequences with Prodigal and then annotates the resulting coding sequences: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs"
- [methods] Traitar requires Pfam 27.0 HMM models downloaded separately: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [discussion] Traitar predicts 67 phenotypes from protein family profiles: "We have developed Traitar, a software framework for predicting phenotypes from the protein family profiles of bacterial genomes."
- [abstract] Traitar is applicable to tens or hundreds of genomes, reducing wet lab work: "It is applicable to studies with tens or hundreds of bacterial genomes. Traitar may help researchers in microbiology to pinpoint the traits of interest"
