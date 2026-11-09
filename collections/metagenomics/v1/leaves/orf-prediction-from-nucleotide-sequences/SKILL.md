---
name: orf-prediction-from-nucleotide-sequences
description: Use when input is raw nucleotide FASTA genome sequence(s) and gene prediction
  has not been performed externally, so ORF calling must precede protein family annotation
  and phenotype assignment.
when_to_use_negative:
- Gene prediction has already been conducted externally — use the Traitar 'from_genes'
  option, which starts at Pfam annotation
- Input consists of protein sequences rather than nucleotide FASTAs
- A feature/Pfam profile table is already available and only phenotype assignment
  is needed
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: Traitar
  role: Pipeline driver orchestrating Prodigal ORF calling, HMMer/Pfam annotation,
    and phenotype prediction via the from_nucleotides workflow
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from input nucleotide FASTA genomes within the
    Traitar from_nucleotides workflow
- name: HMMER
  role: Scans Prodigal-predicted proteins against Pfam 27.0 HMM models to produce
    protein family profiles
- name: Docker (aweimann/traitar)
  role: Containerized execution of the from_nucleotides workflow without local installation
  repo: https://hub.docker.com/r/aweimann/traitar
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:21f297ac1364491c35c8a0d417875fc0e925af5d48ee5f6dc8f9aac1461be4fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Docker (aweimann/traitar)
  - HMMER
  - Prodigal
  - Traitar
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# orf-prediction-from-nucleotide-sequences

## Summary

Predicts protein-coding open reading frames from bacterial nucleotide FASTA genomes using Prodigal as the first step of the Traitar from_nucleotides workflow, producing translated protein sequences for downstream Pfam/HMMer annotation and 67-trait phenotype prediction.

## When to use

Use when input is raw nucleotide FASTA genome sequence(s) and gene prediction has not been performed externally, so ORF calling must precede protein family annotation and phenotype assignment.

## When NOT to use

- Gene prediction has already been conducted externally — use the Traitar 'from_genes' option, which starts at Pfam annotation
- Input consists of protein sequences rather than nucleotide FASTAs
- A feature/Pfam profile table is already available and only phenotype assignment is needed

## Inputs

- Nucleotide FASTA file(s) of bacterial genome(s) provided in the input directory
- Sample file (TSV) with mandatory header and columns: sample_file_name, sample_name, optional category

## Outputs

- Gene prediction results in <out_dir>/gene_prediction (Prodigal-predicted ORFs and translated protein sequences)
- Pfam annotation in <out_dir>/pfam_annotation
- Phenotype prediction tables (e.g. phenotype_predictions.tsv / predictions_majority-vote_combined.txt) and heatmaps in <out_dir>/phenotype_prediction

## How to apply

Run the Traitar 'phenotype ... from_nucleotides' workflow, which first predicts open reading frames from each input nucleotide FASTA with Prodigal and writes the translated protein sequences to <out_dir>/gene_prediction. Traitar then annotates these coding sequences for all samples listed in the sample file with Pfam families using HMMer, applying traitar thresholds of bit-score >= 25 and E value <= 1e-2 against the Pfam 27.0 HMM models, before phenotype prediction with the pre-trained SVM models for the 67 traits. A sample file mapping sample file names to user-defined sample names (and optionally a category column) is required. If gene prediction has already been done externally, use 'from_genes' instead, which starts at the Pfam annotation step. Parallelism can be enabled with the -c flag (requires GNU parallel), though the Docker container has known problems with parallel usage.

## Related tools

- **Traitar** (Pipeline driver orchestrating Prodigal ORF calling, HMMer/Pfam annotation, and phenotype prediction via the from_nucleotides workflow) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from input nucleotide FASTA genomes within the Traitar from_nucleotides workflow)
- **HMMER** (Scans Prodigal-predicted proteins against Pfam 27.0 HMM models to produce protein family profiles)
- **Docker (aweimann/traitar)** (Containerized execution of the from_nucleotides workflow without local installation) — https://hub.docker.com/r/aweimann/traitar

## Examples

```
traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- phenotype_predictions.tsv (or predictions_majority-vote_combined.txt) exists and contains one row per input sample with columns covering the 67 traitar phenotype labels
- <out_dir>/gene_prediction directory is populated with Prodigal gene calls and translated proteins for every sample in the sample file
- <out_dir>/pfam_annotation directory contains Pfam hits obtained with bit-score >= 25 and E value <= 1e-2 against Pfam 27.0
- For from_nucleotides runs, GFF files in phypat/feat_gffs link Prodigal-called genes with the important protein families
- Heatmap files (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) are generated from the prediction tables

## Limitations

- Docker execution has a known problem with parallel usage, so the -c option is not guaranteed to work, and output files are owned by root
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome
- The from_genes feature-track option is experimental; gene ID consistency across gene GFF sources (img, RefSeq, Prodigal) is not guaranteed even with the -g/--gene_gff_type parameter
- Pfam 27.0 HMM models are not distributed with the package and must be downloaded separately via 'traitar pfam'

## Evidence

- [readme] Prodigal ORF calling precedes HMMer/Pfam annotation in from_nucleotides: "which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer"
- [methods] Traitar HMMer filter thresholds for Pfam profile calls: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] Output directories for each pipeline stage: [evidence span withheld — anchor status: external_doc]
- [readme] from_genes alternative when prediction already done: [evidence span withheld — anchor status: external_doc]
- [readme] Sample file format requirements: "The sample file has one column for the sample file names and one for the names as specified by the user. You can also specify a grouping of the samples in the third column"
