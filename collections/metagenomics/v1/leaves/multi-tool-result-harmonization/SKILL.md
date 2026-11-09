---
name: multi-tool-result-harmonization
description: Use when multiple viral-detection tools (VirSorter2, VIBRANT, DeepVirFinder,
  CheckV, MMseqs2 taxonomy) have been run on the same assembled contig library and
  per-contig viral calls must be reconciled into a single confidence-scored table.
when_to_use_negative:
- Input is raw reads or FASTQ rather than an assembled contig library — detection
  must run on contigs first
- Only one detection tool's output is available — the ≥2-criteria confidence rule
  cannot be applied
- The goal is read-level virus detection or assembly itself rather than reconciling
  existing per-contig tool predictions
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: VirSorter2
  role: 'Detection criterion 1: viral category assignment per contig'
  repo: https://github.com/jiarong/VirSorter2
- name: VIBRANT
  role: 'Detection criterion: viral prediction per contig'
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: DeepVirFinder
  role: 'Detection criterion: deep-learning viral prediction with score > 0.9 and
    p < 0.01'
  repo: https://github.com/jessieren/DeepVirFinder
- name: CheckV
  role: 'Detection criterion: viral quality class per contig'
- name: MMseqs2
  role: 'Detection criterion: viral taxonomy hit from annotation against VOGDB/databases'
  repo: https://github.com/soedinglab/MMseqs2
- name: ViroProfiler
  role: Containerized Nextflow pipeline in which the harmonization and confidence
    scoring is executed
  repo: https://github.com/deng-lab/viroprofiler
- name: metaSPAdes
  role: Produces the assembled contig library that is the substrate for all per-tool
    detections
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:4750f396a1e2d3d437a2f14c38310d9fb09b49d02abd3ff92e8356ad1857d1bd
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
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-tool-result-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes contig identifiers across the outputs of multiple viral-detection tools used in ViroProfiler (VirSorter2, VIBRANT, DeepVirFinder, CheckV, plus MMseqs2 taxonomy annotation) and applies a confidence-scoring rule to classify viral contigs as 'high confident' (≥2 tools) or 'low confident' (exactly 1 tool).

## When to use

Use when multiple viral-detection tools (VirSorter2, VIBRANT, DeepVirFinder, CheckV, MMseqs2 taxonomy) have been run on the same assembled contig library and per-contig viral calls must be reconciled into a single confidence-scored table.

## When NOT to use

- Input is raw reads or FASTQ rather than an assembled contig library — detection must run on contigs first
- Only one detection tool's output is available — the ≥2-criteria confidence rule cannot be applied
- The goal is read-level virus detection or assembly itself rather than reconciling existing per-contig tool predictions

## Inputs

- Non-redundant assembled contig library from metaSPAdes (--meta) or bins from vRhyme/phamb
- VirSorter2 viral-category output
- DeepVirFinder score/p-value output (threshold: score > 0.9, p < 0.01)
- VIBRANT viral prediction output
- CheckV viral quality-class output
- MMseqs2 taxonomy annotation output

## Outputs

- viral_contig_confidence_table.tsv: tab-separated table with contig ID, list of detecting tools, and confidence class ('high confident' or 'low confident')

## How to apply

Load the assembled contig library (produced by metaSPAdes with ext.args = '--meta') together with the per-tool detection outputs. Harmonize contig identifiers across the five tool outputs so each contig can be traced through every criterion, accounting for differing identifier conventions between VirSorter2, DeepVirFinder, VIBRANT, CheckV, and MMseqs2 annotation. For each contig, record which of the five criteria detect it as viral: VirSorter2 viral category, MMseqs2 viral taxonomy hit, CheckV viral quality class, DeepVirFinder score > 0.9 with p < 0.01, and VIBRANT viral prediction. Count the number of criteria met per contig and assign 'high confident' to contigs meeting two or more criteria and 'low confident' to contigs meeting exactly one. Write the final result as a tab-separated file (viral_contig_confidence_table.tsv) with columns for contig ID, the list of detecting tools, and the confidence class, ensuring every input contig appears exactly once.

## Related tools

- **VirSorter2** (Detection criterion 1: viral category assignment per contig) — https://github.com/jiarong/VirSorter2
- **VIBRANT** (Detection criterion: viral prediction per contig) — https://github.com/AnantharamanLab/VIBRANT
- **DeepVirFinder** (Detection criterion: deep-learning viral prediction with score > 0.9 and p < 0.01) — https://github.com/jessieren/DeepVirFinder
- **CheckV** (Detection criterion: viral quality class per contig)
- **MMseqs2** (Detection criterion: viral taxonomy hit from annotation against VOGDB/databases) — https://github.com/soedinglab/MMseqs2
- **ViroProfiler** (Containerized Nextflow pipeline in which the harmonization and confidence scoring is executed) — https://github.com/deng-lab/viroprofiler
- **metaSPAdes** (Produces the assembled contig library that is the substrate for all per-tool detections) — https://github.com/ablab/spades

## Evaluation signals

- Every contig in the input contig library appears exactly once in viral_contig_confidence_table.tsv (row_count_equals check)
- Each contig carries a valid label: exactly 'high confident' or 'low confident' (field_present check)
- The per-contig criteria list is consistent with the underlying per-tool outputs (cross-check each detecting tool's report)
- Contigs labeled 'high confident' meet ≥2 of the 5 criteria; 'low confident' contigs meet exactly 1
- DeepVirFinder-derived calls respect the score > 0.9 / p < 0.01 threshold

## Limitations

- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision of the viral calls being harmonized
- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; harmonization quality degrades on samples not enriched for viral fractions
- The confidence rule is a voting heuristic over tool outputs — a contig missed by all but one tool is downgraded to 'low confident' even if genuinely viral

## Evidence

- [readme] DeepVirFinder predicts viral sequences using deep learning method.: [evidence span withheld — anchor status: external_doc]
- [readme] vRhyme is a multi-functional tool for binning virus genomes from metagenomes.: [evidence span withheld — anchor status: external_doc]
