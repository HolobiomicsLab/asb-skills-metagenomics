---
name: protein-family-profile-generation
description: Use when nucleotide (or protein) FASTA genomes must be converted into
  Pfam protein-family profiles as input for phenotype classification, e.g. Traitar
  phypat/phypat+PGL workflows.
when_to_use_negative:
- Genes are already predicted externally and only Pfam annotation is needed — use
  the 'from_genes' workflow instead of re-running Prodigal
- Pfam 27.0 models are unavailable and no download/network access exists — profiles
  cannot be built without the models
- The goal is direct trait prediction from assembled genes with consistent external
  GFFs rather than profile generation itself
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0080
tools:
- name: Traitar
  role: Orchestrates the from_nucleotides workflow that builds the protein-family
    profiles
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from input nucleotide sequences
- name: HMMER
  role: hmmsearch scans predicted proteins against Pfam 27.0 HMM models to call protein
    families
- name: pip
  role: Recommended installer for traitar ('pip install traitar-<version>.tar.gz --user')
  repo: https://github.com/hzi-bifo/traitar
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:3530b043898bc0fb9e2c9ee428069b1aa160097a9c4b749d1cdec52fd49fc81e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER
  - Prodigal
  - Python
  - Python 2.7
  - Traitar
  - hmmer/hmmsearch
  - pip
  - prodigal
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# protein-family-profile-generation

## Summary

Generates binary presence/absence protein-family (Pfam) profile vectors for bacterial genomes by predicting ORFs with Prodigal and scanning proteins against Pfam 27.0 HMMs with HMMER hmmsearch, the feature representation Traitar uses for all 67-phenotype prediction.

## When to use

Use when nucleotide (or protein) FASTA genomes must be converted into Pfam protein-family profiles as input for phenotype classification, e.g. Traitar phypat/phypat+PGL workflows.

## When NOT to use

- Genes are already predicted externally and only Pfam annotation is needed — use the 'from_genes' workflow instead of re-running Prodigal
- Pfam 27.0 models are unavailable and no download/network access exists — profiles cannot be built without the models
- The goal is direct trait prediction from assembled genes with consistent external GFFs rather than profile generation itself

## Inputs

- Nucleotide FASTA file(s) of bacterial genome assemblies (one per sample)
- Sample file with columns sample_file_name, sample_name, and optional category (tab-separated, mandatory header row)
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded via 'traitar pfam' or supplied locally

## Outputs

- Predicted ORFs / translated protein sequences per sample (<out_dir>/gene_prediction)
- Pfam annotation per sample (<out_dir>/pfam_annotation)
- Binary presence/absence protein-family profile vectors per sample, consumed by the 67-trait phenotype prediction (<out_dir>/phenotype prediction)

## How to apply

Start from nucleotide FASTA files for each sample and run Traitar's standard from_nucleotides workflow: Prodigal predicts open reading frames, then the resulting coding sequences are annotated with Pfam families using HMMER hmmsearch against the Pfam 27.0 (Pfam-A.hmm) models that must be fetched beforehand via 'traitar pfam <folder>' or registered locally with 'traitar pfam --local'. Filter HMMER hits using Traitar's thresholds of bit-score >= 25 and E value <= 1e-2, then build a binary presence/absence protein-family profile vector per sample. These profiles feed the pre-trained linear L1-regularized SVM voting committees that assign the 67 phenotype labels (phypat uses phyletic patterns only; phypat+PGL additionally encodes ancestral protein family gains/losses). Use GNU parallel via the '-c' option to parallelize across samples; the Pfam annotation is written to <out_dir>/pfam_annotation and profiles are consumed downstream by the phenotype prediction module.

## Related tools

- **Traitar** (Orchestrates the from_nucleotides workflow that builds the protein-family profiles) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from input nucleotide sequences)
- **HMMER** (hmmsearch scans predicted proteins against Pfam 27.0 HMM models to call protein families)
- **pip** (Recommended installer for traitar ('pip install traitar-<version>.tar.gz --user')) — https://github.com/hzi-bifo/traitar

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- The Pfam models step succeeded: 'traitar pfam <download_folder>' placed Pfam-A.hmm in the target folder, or --local registration points to a valid Pfam 27.0 folder
- <out_dir>/gene_prediction and <out_dir>/pfam_annotation directories exist and contain per-sample results for every entry in the sample file
- Filtered hits respect Traitar's thresholds: bit-score >= 25 and E value <= 1e-2 per called protein family
- Downstream phenotype prediction completes, confirming each profile vector is well-formed for the 67-trait models (e.g. predictions_majority-vote_combined.txt is produced)
- Parallel runs with '-c N' complete without GNU parallel errors and yield profiles identical to a single-process run

## Limitations

- Traitar's genotype-phenotype inference uses only presence/absence of protein families; other omics data are not integrated
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome
- Pfam 27.0 models are not distributed with the package and must be downloaded separately; parallel usage requires GNU parallel (and Docker parallel use is unreliable)

## Evidence

- [readme] the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer: "the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file>"
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [discussion] assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes.: "assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes."
- [readme] Traitar provides the gene prediction results in ``<out_dir>/gene_prediction``, the Pfam annotation in ``<out_dir>/pfam_annotation``: [evidence span withheld — anchor status: external_doc]
- [methods] ``traitar pfam <path to download folder>`` will download and extract the Pfam models to the specified folder.: "``traitar pfam <path to download folder>`` will download and extract the Pfam models to the specified folder."
