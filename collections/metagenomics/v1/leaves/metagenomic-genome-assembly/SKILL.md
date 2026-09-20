---
name: metagenomic-genome-assembly
description: Use when processing shotgun viral metagenomic FASTQ reads in ViroProfiler
  and you need per-sample assembled contigs plus a single species-level non-redundant
  contig library (nrclib.fasta) for downstream annotation, taxonomy assignment, and
  analysis.
when_to_use_negative:
- Input reads are from isolated viral fractions/samples with isolated viruses where
  per-genome assembly rather than metagenomic de novo assembly is appropriate
- Metagenome sequences contain excessive environmental contamination, which the article
  notes increases pipeline running time and lowers precision — clean or isolate viral
  fractions first
- You already have dereplicated contigs or a reference contig library and only need
  annotation or binning (use ViroProfiler's downstream steps, vRhyme, or phamb instead)
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0781
tools:
- name: ViroProfiler
  role: containerized Nextflow pipeline orchestrating assembly, filtering, CheckV
    assessment, and ANI-based dereplication into nrclib
  repo: https://github.com/deng-lab/viroprofiler
- name: metaSPAdes
  role: de novo metagenomic assembly of quality-filtered reads per sample in metagenomic
    mode (--meta)
  repo: https://github.com/ablab/spades
- name: CheckV
  role: 'quality control of assembled viral contigs: evaluate contig quality and completeness'
- name: MGV ani_cluster
  role: ANI-based species-level clustering/dereplication script (>=95% ANI, >=85%
    AF) selecting representative contigs for nrclib
  repo: https://github.com/snayfach/MGV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:ac8697e346f07c151e96fb4097e6da2203cb837fa6717dc40d4a389b1d9fa3c3
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-genome-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble quality-filtered reads from each sample in a viral metagenomic dataset with metaSPAdes in metagenomic mode, merge contigs across samples, filter by minimum length, and dereplicate at the species level (>=95% ANI, >=85% AF per MIUViG) to build the non-redundant contig library (nrclib) used for downstream annotation in ViroProfiler.

## When to use

Use when processing shotgun viral metagenomic FASTQ reads in ViroProfiler and you need per-sample assembled contigs plus a single species-level non-redundant contig library (nrclib.fasta) for downstream annotation, taxonomy assignment, and analysis.

## When NOT to use

- Input reads are from isolated viral fractions/samples with isolated viruses where per-genome assembly rather than metagenomic de novo assembly is appropriate
- Metagenome sequences contain excessive environmental contamination, which the article notes increases pipeline running time and lowers precision — clean or isolate viral fractions first
- You already have dereplicated contigs or a reference contig library and only need annotation or binning (use ViroProfiler's downstream steps, vRhyme, or phamb instead)

## Inputs

- quality-filtered shotgun metagenomic reads per sample (FASTQ)
- merged contig pool from all samples
- ViroProfiler default Nextflow config (CPU/memory resource settings)

## Outputs

- per-sample assembly contigs (metaSPAdes output)
- CheckV quality/completeness assessment of contigs
- species-level ANI/AF clusters (>=95% ANI, >=85% AF)
- non-redundant contig library FASTA (nrclib.fasta) with one representative per cluster

## How to apply

Assemble quality-filtered reads for each sample with metaSPAdes run in metagenomic mode (ext.args = '--meta'), allocated 4 CPUs and 100 GB memory per the pipeline's default Nextflow config. Merge the contigs from all samples into a single pool and filter by minimum length to remove short spurious sequences. Assess pooled contigs with CheckV (1 CPU, 30 GB memory) to evaluate viral contig quality and completeness. Dereplicate the long contig library using the MGV ani_cluster script, clustering sequences at the species level with >=95% average nucleotide identity (ANI) and >=85% alignment fraction (AF), consistent with MIUViG guidelines; each cluster approximates a viral species. Write one representative contig per species-level cluster into nrclib.fasta, which ViroProfiler then feeds into downstream annotation and analysis steps.

## Related tools

- **ViroProfiler** (containerized Nextflow pipeline orchestrating assembly, filtering, CheckV assessment, and ANI-based dereplication into nrclib) — https://github.com/deng-lab/viroprofiler
- **metaSPAdes** (de novo metagenomic assembly of quality-filtered reads per sample in metagenomic mode (--meta)) — https://github.com/ablab/spades
- **CheckV** (quality control of assembled viral contigs: evaluate contig quality and completeness)
- **MGV ani_cluster** (ANI-based species-level clustering/dereplication script (>=95% ANI, >=85% AF) selecting representative contigs for nrclib) — https://github.com/snayfach/MGV

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker --input samples.csv --ext.args '--meta'
```

## Evaluation signals

- nrclib.fasta exists and is a valid FASTA file
- nrclib contains exactly one representative sequence per species-level cluster (no duplicate contigs)
- No two contigs in nrclib share >=95% ANI with >=85% AF — the MIUViG dereplication threshold holds
- Per-sample assembly completed with metaSPAdes using ext.args = '--meta' at the configured 4 CPUs / 100 GB memory
- CheckV report exists for the pooled, length-filtered contigs indicating quality/completeness was assessed

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase the pipeline running time and result in lower precision; it is recommended to isolate the viral fractions before sequencing
- Dereplication clusters approximate viral species, so cluster representatives are species-level approximations rather than verified complete genomes

## Evidence

- [readme] MGV code availability including viral detection pipeline and CheckV QC (README): [evidence span withheld — anchor status: not_found]
- [discussion] recommendation to isolate viral fractions before sequencing: [evidence span withheld — anchor status: not_found]
