---
name: contig-length-filtering
description: Use when per-sample contigs assembled by metaSPAdes ('--meta' mode) must
  be merged and length-filtered prior to CheckV assessment and ANI-based dereplication
  into the nrclib in ViroProfiler.
when_to_use_negative:
- Input is already a dereplicated nrclib or a curated viral genome set rather than
  pooled per-sample metagenomic assemblies
- The analysis targets microbial genomes/binning rather than viral contigs for the
  ViroProfiler nrclib
- Samples contain isolated virus sequences rather than metagenomic assemblies — the
  article notes ViroProfiler is specifically designed for classifying viruses in samples
  with isolated viruses, where a length filter on metagenome-assembled contigs is
  not the entry point
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline that merges per-sample contigs, applies the
    minimum-length filter, and dereplicates them into nrclib.fasta
  repo: https://github.com/deng-lab/viroprofiler
- name: metaSPAdes
  role: Per-sample genome assembly in metagenomic mode (ext.args = '--meta') producing
    the contigs that are merged and length-filtered
  repo: https://github.com/ablab/spades
- name: CheckV
  role: Assesses viral contig quality and completeness after length filtering (1 CPU,
    30 GB memory)
- name: MGV ani_cluster
  role: ANI clustering script that dereplicates the filtered long contig library at
    the species level (>=95% ANI, >=85% AF) to yield nrclib
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:3082766ec4290a11b60769603ac4f434eda5cbad989b0234ddbe84b43fcd061f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
  tools:
  - CheckV
  - MGV ani_cluster
  - ViroProfiler
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# contig-length-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters pooled per-sample metagenomic assemblies by a minimum contig length before species-level dereplication, removing short spurious sequences so that only long, assembly-informed contigs feed into ViroProfiler's non-redundant contig library (nrclib).

## When to use

Use when per-sample contigs assembled by metaSPAdes ('--meta' mode) must be merged and length-filtered prior to CheckV assessment and ANI-based dereplication into the nrclib in ViroProfiler.

## When NOT to use

- Input is already a dereplicated nrclib or a curated viral genome set rather than pooled per-sample metagenomic assemblies
- The analysis targets microbial genomes/binning rather than viral contigs for the ViroProfiler nrclib
- Samples contain isolated virus sequences rather than metagenomic assemblies — the article notes ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, where a length filter on metagenome-assembled contigs is not the entry point

## Inputs

- Quality-filtered shotgun metagenomic reads (FASTQ) per sample
- Per-sample metaSPAdes metagenomic-mode assemblies (contig FASTA files)
- Pooled contig FASTA from all samples
- Minimum contig length threshold (pipeline config parameter)

## Outputs

- Length-filtered (long) contig library FASTA
- CheckV quality/completeness assessment of filtered contigs
- Species-level dereplicated non-redundant contig library (nrclib.fasta) with one representative per >=95% ANI / >=85% AF cluster

## How to apply

Assemble quality-filtered reads for each sample with metaSPAdes in metagenomic mode (ext.args = '--meta', 4 CPUs / 100 GB memory in the pipeline's default config), then merge contigs from all samples into a single pool. Apply the minimum-length filter to this pool to discard short spurious sequences that would inflate the dereplication space. Pass the filtered long contigs to CheckV (1 CPU, 30 GB) to assess viral contig quality and completeness, then cluster at >=95% average nucleotide identity (ANI) and >=85% alignment fraction (AF), consistent with MIUViG guidelines, using the MGV ani_cluster script. Write one representative contig per species-level cluster to nrclib.fasta for downstream annotation and analysis.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline that merges per-sample contigs, applies the minimum-length filter, and dereplicates them into nrclib.fasta) — https://github.com/deng-lab/viroprofiler
- **metaSPAdes** (Per-sample genome assembly in metagenomic mode (ext.args = '--meta') producing the contigs that are merged and length-filtered) — https://github.com/ablab/spades
- **CheckV** (Assesses viral contig quality and completeness after length filtering (1 CPU, 30 GB memory))
- **MGV ani_cluster** (ANI clustering script that dereplicates the filtered long contig library at the species level (>=95% ANI, >=85% AF) to yield nrclib)

## Evaluation signals

- nrclib.fasta exists and is a valid FASTA file
- nrclib.fasta contains exactly one representative sequence per species-level cluster
- No duplicate contigs remain at the >=95% ANI / >=85% AF dereplication threshold
- All contigs in the filtered library meet the configured minimum length, with short spurious sequences removed before CheckV and clustering
- metaSPAdes ran with metagenomic mode ('--meta') and CheckV quality reports were produced for the filtered contigs

## Limitations

- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision, motivating viral fraction isolation before sequencing
- The minimum length threshold and dereplication parameters are pipeline-configured; changing them changes which sequences reach the nrclib and downstream annotation
- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, not for whole-community microbial assembly filtering

## Evidence

- [methods] Genome assembly | [metaSPAdes](https://github.com/ablab/spades): "Genome assembly | [metaSPAdes](https://github.com/ablab/spades)"
