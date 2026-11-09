---
name: hmm-profile-annotation-against-pfam
description: Use when starting from nucleotide FASTA genomes (or already-predicted
  ORFs) and you need Pfam-A protein family profiles as features for Traitar phenotype
  prediction across tens or hundreds of bacterial samples.
when_to_use_negative:
- Input consists of raw reads or transcriptomic/proteomic data — Traitar only takes
  nucleotide or protein sequences of genomes and uses only protein family presence/absence,
  not other omics data
- Input is already an annotated Pfam domain table — in that case proceed directly
  to Traitar's phenotype-prediction stage rather than re-running Prodigal/HMMer
- Only a handful of traits tied to specific curated gene sets are needed and a genotype-phenotype
  association tool like PICA is more appropriate for the question
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3301
tools:
- name: Traitar
  role: 'Orchestrates the pipeline: downloads/registers Pfam models, runs Prodigal
    and HMMER annotation, and converts Pfam hits into phenotype predictions for 67
    traits'
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from the input nucleotide FASTAs and writes translated
    protein sequences for downstream Pfam annotation
- name: HMMER
  role: Runs hmmsearch of predicted proteins against the Pfam 27.0 (Pfam-A) HMM models
    with Traitar's bit-score and E-value cutoffs
- name: Pfam 27.0 (Pfam-A.hmm)
  role: The HMM profile database that defines the protein families used as classifier
    features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:10ba235efb95a2555de5f53ad336556e3557582e32c352d81d23af605dc06f88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - HMMER
  - Pfam
  - Pfam 27.0 (Pfam-A.hmm)
  - Prodigal
  - Traitar
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# HMM profile annotation against Pfam

## Summary

Annotate predicted protein-coding sequences from microbial genomes with Pfam protein family profiles using HMMER hmmsearch, as implemented in the Traitar microbial trait analyzer, to build the binary presence/absence protein-family profiles that feed its 67-phenotype SVM classifiers.

## When to use

Use when starting from nucleotide FASTA genomes (or already-predicted ORFs) and you need Pfam-A protein family profiles as features for Traitar phenotype prediction across tens or hundreds of bacterial samples.

## When NOT to use

- Input consists of raw reads or transcriptomic/proteomic data — Traitar only takes nucleotide or protein sequences of genomes and uses only protein family presence/absence, not other omics data
- Input is already an annotated Pfam domain table — in that case proceed directly to Traitar's phenotype-prediction stage rather than re-running Prodigal/HMMer
- Only a handful of traits tied to specific curated gene sets are needed and a genotype-phenotype association tool like PICA is more appropriate for the question

## Inputs

- Nucleotide FASTA file(s) of bacterial genome(s) (from_nucleotides mode) or externally gene-predicted coding sequences (from_genes mode)
- Sample file (TSV with columns sample_file_name, sample_name, optional category) listing all samples to annotate
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded or registered locally via traitar pfam

## Outputs

- Predicted protein sequences / gene predictions in <out_dir>/gene_prediction
- Pfam protein family annotation per sample in <out_dir>/pfam_annotation
- Binary presence/absence protein-family profile vectors used by the phenotype classifiers
- Downstream phenotype prediction tables and heatmaps in <out_dir>/phenotype_prediction (e.g. predictions_majority-vote_combined.txt, heatmap_phypat.png)

## How to apply

First predict protein-coding open reading frames from each input nucleotide FASTA with Prodigal (skip this if starting from translated genes via the from_genes mode). Scan the predicted proteins against the Pfam 27.0 HMM models (Pfam-A.hmm), which Traitar requires to be downloaded separately via 'traitar pfam <folder>' or registered with 'traitar pfam --local <path>', using HMMER hmmsearch with Traitar's thresholds of bit-score >= 25 and E value <= 1e-2 to call protein family profiles. Convert the filtered Pfam hits into a binary presence/absence protein-family profile vector per sample. These profiles are the features consumed by the pre-trained L1-regularized linear SVM voting committees that assign the 67 phenotype labels; results land in <out_dir>/pfam_annotation and feed <out_dir>/phenotype_prediction. Parallelize across samples with the -c flag (requires GNU parallel) when many genomes are annotated.

## Related tools

- **Traitar** (Orchestrates the pipeline: downloads/registers Pfam models, runs Prodigal and HMMER annotation, and converts Pfam hits into phenotype predictions for 67 traits) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from the input nucleotide FASTAs and writes translated protein sequences for downstream Pfam annotation)
- **HMMER** (Runs hmmsearch of predicted proteins against the Pfam 27.0 (Pfam-A) HMM models with Traitar's bit-score and E-value cutoffs)
- **Pfam 27.0 (Pfam-A.hmm)** (The HMM profile database that defines the protein families used as classifier features)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- phenotype_predictions.tsv (or predictions_majority-vote_combined.txt) exists and contains one row per input sample with columns covering the 67 Traitar phenotype labels
- <out_dir>/gene_prediction and <out_dir>/pfam_annotation directories are populated for every sample listed in the sample file
- Pfam hits passed Traitar thresholds: reported profile matches have bit-score >= 25 and E value <= 1e-2
- Presence/absence profile vectors are binary and one vector per sample was built from the filtered Pfam hits
- Encoding of combined predictions is consistent with the README convention (0 = negative, 1 = phypat-only, 2 = phylogeny-aware only, 3 = supported by both)

## Limitations

- Pfam 27.0 HMM models are not distributed with the Traitar package and must be downloaded or registered locally before annotation
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome rather than true trait absence
- Genotype-phenotype inference uses only presence/absence of protein families; other omics data are not integrated
- Parallel execution via -c requires GNU parallel, and the README notes parallel usage is not guaranteed to work inside the Docker container

## Evidence

- [readme] Standard Traitar from_nucleotides workflow: "predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer"
- [methods] Pfam 27.0 requirement and setup: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [methods] Pfam model registration command: "``traitar pfam <path to download folder>`` will download and extract the Pfam models to the specified folder."
- [other] Thresholds for Pfam profile calling: [evidence span withheld — anchor status: not_found]
- [readme] from_genes alternative entry point: [evidence span withheld — anchor status: external_doc]
- [discussion] Incomplete genome caveat: "the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes."
