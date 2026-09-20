---
name: metagenomic-contig-taxonomy-annotation
description: Use when you need to classify assembled metagenomic contigs as putative
  viral sequences and need a reproducible confidence label based on agreement across
  multiple detection tools rather than a single-tool call.
when_to_use_negative:
- Input is raw reads rather than an assembled contig library — the scoring rule operates
  on assembled contigs, not FASTQ
- Samples contain excessive environmental contamination — the paper warns this can
  increase runtime and lower precision, and recommends isolating viral fractions before
  sequencing
- You only need single-tool viral prediction without cross-tool confidence labels
  — running one detector alone is simpler and this skill's value is the multi-tool
  consensus rule
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: VirSorter2
  role: Viral sequence detection; its viral-category call is one confidence criterion
  repo: https://github.com/jiarong/VirSorter2
- name: VIBRANT
  role: Viral prediction; its per-contig viral call is one confidence criterion
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: DeepVirFinder
  role: Deep-learning viral detection; score > 0.9 and p < 0.01 is one confidence
    criterion
  repo: https://github.com/jessieren/DeepVirFinder
- name: CheckV
  role: Viral quality assessment; its viral quality class is one confidence criterion
  repo: https://bitbucket.org/berkeleylab/checkv
- name: MMseqs2
  role: Taxonomy annotation of contigs; a viral taxonomy hit is one confidence criterion
  repo: https://github.com/soedinglab/MMseqs2
- name: ViroProfiler
  role: Containerized pipeline orchestrating the detection tools and the confidence-scoring
    rule
  repo: https://github.com/deng-lab/viroprofiler
- name: metaSPAdes
  role: Metatranscriptome/metagenome assembler producing the contig library with ext.args
    = '--meta'
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:935211681717f43dd493e877ea07f8e32250cbd373f0d8067dbd075d4059e8a4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/jiarong/VirSorter2
  tools:
  - CheckV
  - DeepVirFinder
  - MMseqs2
  - MMseqs2 taxonomy
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# metagenomic-contig-taxonomy-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs ViroProfiler's viral contig identification stage, in which the assembled contig library is screened by multiple viral-detection tools and a multi-tool confidence scoring rule (>=2 criteria = high confident, exactly 1 = low confident) assigns a confidence class per contig.

## When to use

Use when you need to classify assembled metagenomic contigs as putative viral sequences and need a reproducible confidence label based on agreement across multiple detection tools rather than a single-tool call.

## When NOT to use

- Input is raw reads rather than an assembled contig library — the scoring rule operates on assembled contigs, not FASTQ
- Samples contain excessive environmental contamination — the paper warns this can increase runtime and lower precision, and recommends isolating viral fractions before sequencing
- You only need single-tool viral prediction without cross-tool confidence labels — running one detector alone is simpler and this skill's value is the multi-tool consensus rule

## Inputs

- Non-redundant assembled contig library (metaSPAdes --meta FASTA)
- VirSorter2 viral-category output per contig
- VIBRANT viral prediction output per contig
- DeepVirFinder score and p-value table per contig
- CheckV viral quality-class output per contig
- MMseqs2 taxonomy annotation hits per contig

## Outputs

- viral_contig_confidence_table.tsv with columns: contig ID, list of detecting tools, confidence class ('high confident' or 'low confident')

## How to apply

Load the non-redundant contig library assembled by metaSPAdes (run with ext.args = '--meta') together with the per-tool outputs from VirSorter2, VIBRANT, DeepVirFinder, CheckV, and MMseqs2 taxonomy annotation. Harmonize contig identifiers so each contig can be traced through every criterion. For each contig, record which criteria detect it as viral: VirSorter2 viral category, MMseqs2 viral taxonomy hit, CheckV viral quality class, DeepVirFinder score > 0.9 with p < 0.01, and VIBRANT viral prediction. Count criteria met per contig and assign 'high confident' when two or more criteria fire and 'low confident' when exactly one fires. Write the result as a tab-separated table with contig ID, detecting tools, and confidence class.

## Related tools

- **VirSorter2** (Viral sequence detection; its viral-category call is one confidence criterion) — https://github.com/jiarong/VirSorter2
- **VIBRANT** (Viral prediction; its per-contig viral call is one confidence criterion) — https://github.com/AnantharamanLab/VIBRANT
- **DeepVirFinder** (Deep-learning viral detection; score > 0.9 and p < 0.01 is one confidence criterion) — https://github.com/jessieren/DeepVirFinder
- **CheckV** (Viral quality assessment; its viral quality class is one confidence criterion) — https://bitbucket.org/berkeleylab/checkv
- **MMseqs2** (Taxonomy annotation of contigs; a viral taxonomy hit is one confidence criterion) — https://github.com/soedinglab/MMseqs2
- **ViroProfiler** (Containerized pipeline orchestrating the detection tools and the confidence-scoring rule) — https://github.com/deng-lab/viroprofiler
- **metaSPAdes** (Metatranscriptome/metagenome assembler producing the contig library with ext.args = '--meta') — https://github.com/ablab/spades

## Evaluation signals

- Every contig in the input library appears exactly once in viral_contig_confidence_table.tsv (row_count_equals check)
- Every output contig carries a valid 'high confident' or 'low confident' label (field_present check)
- The criteria list per contig is consistent with the underlying per-tool outputs for VirSorter2, VIBRANT, DeepVirFinder, CheckV, and MMseqs2
- Confidence rule holds: 'high confident' rows meet >= 2 criteria, 'low confident' rows meet exactly 1
- DeepVirFinder-based labels only appear for contigs with score > 0.9 and p < 0.01

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase the pipeline running time and result in lower precision
- It is recommended to isolate the viral fractions before sequencing for an accurate estimation of viruses in the samples

## Evidence

- [readme] MMseqs2 is a software suite to search and cluster huge protein and nucleotide sequence sets.: [evidence span withheld — anchor status: external_doc]
