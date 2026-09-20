---
name: viral-contig-identification
description: Use when you have a non-redundant contig library (e.g. metaSPAdes --meta
  output) from viral metagenomic data and need to decide which contigs are viral and
  how confident those calls are.
when_to_use_negative:
- Input is raw reads or isolated viral fractions requiring no contig-level viral/non-viral
  discrimination — ViroProfiler is specifically designed for classifying viruses in
  metagenomic assemblies, not samples with isolated viruses
- Sequences carry excessive environmental contamination, which the paper warns increases
  running time and lowers precision
- Only a single viral detector's output is available and no multi-tool evidence can
  be gathered to support the confidence scoring
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: ViroProfiler
  role: Nextflow pipeline that orchestrates viral contig identification and the multi-tool
    confidence scoring
  repo: https://github.com/deng-lab/viroprofiler
- name: VirSorter2
  role: Detects viral contigs via viral category classification
  repo: https://github.com/jiarong/VirSorter2
- name: DeepVirFinder
  role: Deep-learning viral detection; contig counts as viral when score > 0.9 and
    p < 0.01
  repo: https://github.com/jessieren/DeepVirFinder
- name: VIBRANT
  role: Viral prediction contributing the fifth criterion to the confidence scoring
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: CheckV
  role: Assesses viral quality class of contigs, contributing a detection criterion
- name: MMseqs2
  role: Taxonomy annotation; a viral taxonomy hit counts as one detection criterion
  repo: https://github.com/soedinglab/MMseqs2
- name: metaSPAdes
  role: De novo metagenomic assembly producing the contig library (ext.args = '--meta')
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_007
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:f4170b8e1e2accd4d5188630551f0541003dc7265f88a95d7e988d0d01e8d7b3
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
  - BACPHLIP
  - CheckV
  - DeepVirFinder
  - FASTP
  - MMseqs2
  - Replidec
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - iPhoP
  - metaSPAdes
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# viral-contig-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify putative viral contigs in shotgun virome assemblies by combining multiple detection tools (VirSorter2, VIBRANT, DeepVirFinder, CheckV, MMseqs2 taxonomy) and classifying contigs by the number of tools that detect them, as implemented in the ViroProfiler pipeline.

## When to use

Use when you have a non-redundant contig library (e.g. metaSPAdes --meta output) from viral metagenomic data and need to decide which contigs are viral and how confident those calls are.

## When NOT to use

- Input is raw reads or isolated viral fractions requiring no contig-level viral/non-viral discrimination — ViroProfiler is specifically designed for classifying viruses in metagenomic assemblies, not samples with isolated viruses
- Sequences carry excessive environmental contamination, which the paper warns increases running time and lowers precision
- Only a single viral detector's output is available and no multi-tool evidence can be gathered to support the confidence scoring

## Inputs

- Non-redundant assembled contig library from metaSPAdes (--meta) or bins from vRhyme/phamb
- Per-tool detection outputs from VirSorter2, DeepVirFinder, VIBRANT, CheckV, and MMseqs2 taxonomy annotation

## Outputs

- viral_contig_confidence_table.tsv with columns: contig ID, list of detecting tools, confidence class ('high confident' or 'low confident')
- Set of classified viral contigs passed downstream to taxonomy assignment, host prediction, and replication cycle prediction

## How to apply

Assemble reads with metaSPAdes (--meta) into a non-redundant contig library, then run four viral detectors — VirSorter2, VIBRANT, DeepVirFinder, and CheckV — plus MMseqs2 taxonomy annotation on the contigs or bins. Harmonize contig identifiers across the five tool outputs so each contig is traceable through every criterion. For each contig record which criteria fire: VirSorter2 viral category, MMseqs2 viral taxonomy hit, CheckV viral quality class, DeepVirFinder score > 0.9 with p < 0.01, and VIBRANT viral prediction. Assign 'high confident' to contigs meeting two or more criteria and 'low confident' to contigs meeting exactly one, since ViroProfiler applies a multi-tool scoring system rather than trusting any single detector. Output a tab-separated table with contig ID, the list of detecting tools, and the confidence class.

## Related tools

- **ViroProfiler** (Nextflow pipeline that orchestrates viral contig identification and the multi-tool confidence scoring) — https://github.com/deng-lab/viroprofiler
- **VirSorter2** (Detects viral contigs via viral category classification) — https://github.com/jiarong/VirSorter2
- **DeepVirFinder** (Deep-learning viral detection; contig counts as viral when score > 0.9 and p < 0.01) — https://github.com/jessieren/DeepVirFinder
- **VIBRANT** (Viral prediction contributing the fifth criterion to the confidence scoring) — https://github.com/AnantharamanLab/VIBRANT
- **CheckV** (Assesses viral quality class of contigs, contributing a detection criterion)
- **MMseqs2** (Taxonomy annotation; a viral taxonomy hit counts as one detection criterion) — https://github.com/soedinglab/MMseqs2
- **metaSPAdes** (De novo metagenomic assembly producing the contig library (ext.args = '--meta')) — https://github.com/ablab/spades

## Examples

```
nextflow run deng-lab/viroprofiler -c custom.config --input samples.csv --outdir results
```

## Evaluation signals

- Every contig in the input library appears exactly once in viral_contig_confidence_table.tsv (row_count_equals check)
- Each confidence label is a valid 'high confident' or 'low confident' value (field_present check)
- The criteria list per contig is consistent with the per-tool outputs for VirSorter2, DeepVirFinder (score > 0.9, p < 0.01), VIBRANT, CheckV, and MMseqs2 taxonomy
- 'high confident' contigs meet >= 2 of the five criteria; 'low confident' contigs meet exactly one
- On the UC fecal virome reproduction, the workflow recovered 761 viral contigs vs 183 in the original study, demonstrating improved viral discovery

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, and excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision
- Isolating viral fractions before sequencing is recommended for an accurate estimation of viruses in the samples
- Confidence labels are relative (>=2 criteria vs exactly one); no external benchmark datasets are named for validation in the discussion section

## Evidence

- [results] ViroProfiler combines four detection tools (VirSorter2, VIBRANT, DeepVirFinder, CheckV) on the non-redundant contig library or bins, and applies a scoring system to classify viral contigs identified by multiple tools.: [evidence span withheld — anchor status: not_found]
- [results] DeepVirFinder score > 0.9 and p < 0.01 is one of the five detection criteria.: [evidence span withheld — anchor status: not_found]
