---
name: checkv-quality-class-interpretation
description: Use when a viral metagenomic contig library must be screened for viral
  sequences and CheckV quality-class output should be merged with VirSorter2, VIBRANT,
  DeepVirFinder, and MMseqs2 hits into a per-contig confidence label.
when_to_use_negative:
- The input is raw reads rather than an assembled contig library or bins — detection
  criteria operate on assembled contigs.
- Samples contain isolated (cultured) viruses rather than metagenomic communities,
  since ViroProfiler is designed for classifying viruses in metagenomic samples and
  excessive host contamination lowers precision.
- Only one detection tool's output is available — the confidence rule requires per-contig
  criteria from multiple tools (VirSorter2, VIBRANT, DeepVirFinder, CheckV, MMseqs2).
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: CheckV
  role: Provides the viral quality class per contig used as one of the five viral-detection
    criteria and flags host regions flanking viral contigs
  repo: https://bitbucket.org/berkeleylab/checkv
- name: ViroProfiler
  role: Containerized Nextflow pipeline that runs and merges the detection tools and
    applies the multi-criteria confidence scoring
  repo: https://github.com/deng-lab/viroprofiler
- name: VirSorter2
  role: Supplies the viral-category criterion per contig
  repo: https://github.com/jiarong/VirSorter2
- name: DeepVirFinder
  role: Supplies the deep-learning criterion (score > 0.9, p < 0.01)
  repo: https://github.com/jessieren/DeepVirFinder
- name: VIBRANT
  role: Supplies the iterative-annotation viral prediction criterion
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: MMseqs2
  role: Supplies taxonomy annotation hits used as the viral-taxonomy criterion
  repo: https://github.com/soedinglab/MMseqs2
- name: metaSPAdes
  role: Assembles the metagenomic contig library that all detection criteria are evaluated
    on
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:a8e3857130034e5d10b38c30c7774d0df60cf2c6d43a5037a2c48ce69a3f89a6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  tools:
  - CheckV
  - DeepVirFinder
  - MMseqs2
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# checkv-quality-class-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets CheckV output as one of the multi-tool viral detection criteria inside ViroProfiler, using the CheckV viral quality class as evidence that an assembled contig (produced by metaSPAdes with '--meta') is viral, and combining it with VirSorter2, VIBRANT, DeepVirFinder, and MMseqs2 taxonomy results to assign a high/low confidence class.

## When to use

Use when a viral metagenomic contig library must be screened for viral sequences and CheckV quality-class output should be merged with VirSorter2, VIBRANT, DeepVirFinder, and MMseqs2 hits into a per-contig confidence label.

## When NOT to use

- The input is raw reads rather than an assembled contig library or bins — detection criteria operate on assembled contigs.
- Samples contain isolated (cultured) viruses rather than metagenomic communities, since ViroProfiler is designed for classifying viruses in metagenomic samples and excessive host contamination lowers precision.
- Only one detection tool's output is available — the confidence rule requires per-contig criteria from multiple tools (VirSorter2, VIBRANT, DeepVirFinder, CheckV, MMseqs2).

## Inputs

- Assembled non-redundant contig library (metaSPAdes '--meta' FASTA)
- CheckV output with viral quality classes per contig
- VirSorter2 viral-category output
- DeepVirFinder score/p-value output
- VIBRANT viral prediction output
- MMseqs2 taxonomy annotation output

## Outputs

- viral_contig_confidence_table.tsv with columns: contig ID, list of detecting tools, confidence class ('high confident' or 'low confident')

## How to apply

Load the assembled contig library (metaSPAdes run with ext.args = '--meta') together with per-tool outputs from VirSorter2, DeepVirFinder, VIBRANT, CheckV, and MMseqs2 taxonomy annotation, and harmonize contig identifiers so every contig is traceable across all tools. For each contig, record whether the CheckV viral quality class flags it as viral; treat this as one of five binary criteria alongside the VirSorter2 viral category, an MMseqs2 viral taxonomy hit, a DeepVirFinder score > 0.9 with p < 0.01, and a VIBRANT viral prediction. Count criteria met per contig: two or more criteria yield a 'high confident' label, exactly one yields 'low confident'. The rationale is that CheckV provides genome-quality evidence for candidate viral contigs, so it complements prediction-only tools rather than duplicating them, improving confidence in the final viral call. Emit a tab-separated table (contig ID, detecting tools, confidence class) covering every contig exactly once.

## Related tools

- **CheckV** (Provides the viral quality class per contig used as one of the five viral-detection criteria and flags host regions flanking viral contigs) — https://bitbucket.org/berkeleylab/checkv
- **ViroProfiler** (Containerized Nextflow pipeline that runs and merges the detection tools and applies the multi-criteria confidence scoring) — https://github.com/deng-lab/viroprofiler
- **VirSorter2** (Supplies the viral-category criterion per contig) — https://github.com/jiarong/VirSorter2
- **DeepVirFinder** (Supplies the deep-learning criterion (score > 0.9, p < 0.01)) — https://github.com/jessieren/DeepVirFinder
- **VIBRANT** (Supplies the iterative-annotation viral prediction criterion) — https://github.com/AnantharamanLab/VIBRANT
- **MMseqs2** (Supplies taxonomy annotation hits used as the viral-taxonomy criterion) — https://github.com/soedinglab/MMseqs2
- **metaSPAdes** (Assembles the metagenomic contig library that all detection criteria are evaluated on) — https://github.com/ablab/spades

## Evaluation signals

- Every contig in the input contig library appears exactly once in viral_contig_confidence_table.tsv (row_count_equals check).
- Every row carries a valid label of exactly 'high confident' or 'low confident' (field_present check).
- The per-row list of detecting tools matches the per-tool outputs exactly — CheckV quality class, VirSorter2 category, MMseqs2 hit, DeepVirFinder score > 0.9 & p < 0.01, VIBRANT prediction.
- Label logic invariants: ≥2 criteria → 'high confident'; exactly 1 criterion → 'low confident'; 0 criteria should not appear as viral.
- Harmonized contig identifiers resolve to the same sequence across all five tool outputs (no unmatched IDs).

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; mixed host contamination is a mismatch.
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision — isolating viral fractions before sequencing is recommended.
- A contig flagged by exactly one criterion is only 'low confident', so single-tool CheckV evidence alone should not be treated as a definitive viral call.

## Evidence

- [abstract] ViroProfiler is an in-silico workflow for analyzing shotgun viral metagenomic data: "Here we present ViroProfiler, an in-silico workflow for analyzing shotgun viral metagenomic data."
- [intro] Viral discovery tools include VirSorter2, VIBRANT, DeepVirFinder: "tools designed for viral discovery, which include VirSorter29, VIBRANT10, DeepVirFinder11, and VIP12"
- [methods] Genome assembly uses metaSPAdes: "| Genome assembly | [metaSPAdes](https://github.com/ablab/spades) |"
- [readme] CheckV identifies and removes putative host regions flanking viral contigs: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation: designed for samples with isolated viruses: "However, ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses."
