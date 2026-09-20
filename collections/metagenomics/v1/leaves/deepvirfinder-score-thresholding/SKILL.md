---
name: deepvirfinder-score-thresholding
description: Use when classifying assembled metagenomic contigs as viral in ViroProfiler,
  specifically to apply DeepVirFinder's score/p-value threshold as one of the per-tool
  detection criteria.
when_to_use_negative:
- When ViroProfiler is applied to samples with isolated viruses only — the pipeline
  is specifically designed for classifying viruses in samples with isolated viruses,
  and environmental contamination can lower precision.
- When contigs are already classified (e.g., binned vMAGs from vRhyme or phamb) and
  only annotation or taxonomy is needed.
- When you need a standalone viral verdict — a single DeepVirFinder pass alone yields
  only a 'low confident' label under the multi-tool rule.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: DeepVirFinder
  role: Deep-learning tool whose score/p-value output is thresholded (score > 0.9,
    p < 0.01) as one of the viral detection criteria
  repo: https://github.com/jessieren/DeepVirFinder
- name: VirSorter2
  role: Parallel viral detection tool whose viral category contributes an additional
    criterion in the multi-tool confidence scoring
  repo: https://github.com/jiarong/VirSorter2
- name: VIBRANT
  role: Parallel viral prediction tool contributing another criterion to the confidence
    scoring of viral contigs
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: ViroProfiler
  role: Containerized pipeline that orchestrates the detection tools and applies the
    scoring system to classify viral contigs
  repo: https://github.com/deng-lab/viroprofiler
- name: CheckV
  role: Assesses viral genome quality class, contributing a criterion to the multi-tool
    viral contig classification
  repo: https://bitbucket.org/berkeleylab/checkv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:264e6764d31e7d2977bee3e8cbb9dcbf15d658a016289aec6f1cc32cb91a4ed3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/jessieren/DeepVirFinder
  tools:
  - CheckV
  - DeepVirFinder
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# deepvirfinder-score-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A ViroProfiler decision rule that flags contigs as putatively viral when DeepVirFinder's deep-learning score exceeds 0.9 with p < 0.01, feeding this criterion into the multi-tool confidence scoring used to classify viral contigs.

## When to use

Use when classifying assembled metagenomic contigs as viral in ViroProfiler, specifically to apply DeepVirFinder's score/p-value threshold as one of the per-tool detection criteria.

## When NOT to use

- When ViroProfiler is applied to samples with isolated viruses only — the pipeline is specifically designed for classifying viruses in samples with isolated viruses, and environmental contamination can lower precision.
- When contigs are already classified (e.g., binned vMAGs from vRhyme or phamb) and only annotation or taxonomy is needed.
- When you need a standalone viral verdict — a single DeepVirFinder pass alone yields only a 'low confident' label under the multi-tool rule.

## Inputs

- Non-redundant assembled contig library (FASTA, produced by metaSPAdes with '--meta')
- DeepVirFinder prediction output per contig (score and p-value columns)

## Outputs

- Per-contig binary DeepVirFinder criterion (viral if score > 0.9 and p < 0.01)
- Contribution to viral_contig_confidence_table.tsv (contig ID, list of detecting tools, high/low confident class)

## How to apply

Run DeepVirFinder on the non-redundant contig library assembled by metaSPAdes (metaSPAdes is invoked with ext.args = '--meta'), producing a per-contig score and p-value for each prediction. Harmonize contig identifiers between the DeepVirFinder output and the other detection tools (VirSorter2, VIBRANT, CheckV, MMseqs2 taxonomy) so every contig is traceable across criteria. Apply the threshold of score > 0.9 and p < 0.01 to call each contig 'viral' under the DeepVirFinder criterion. Combine this binary call with the other tool outputs, count how many of the criteria each contig meets, and label contigs meeting two or more criteria 'high confident' and those meeting exactly one 'low confident'. The rationale is that a single deep-learning hit is weak evidence, so the threshold result only contributes one vote to ViroProfiler's multi-tool confidence scoring system.

## Related tools

- **DeepVirFinder** (Deep-learning tool whose score/p-value output is thresholded (score > 0.9, p < 0.01) as one of the viral detection criteria) — https://github.com/jessieren/DeepVirFinder
- **VirSorter2** (Parallel viral detection tool whose viral category contributes an additional criterion in the multi-tool confidence scoring) — https://github.com/jiarong/VirSorter2
- **VIBRANT** (Parallel viral prediction tool contributing another criterion to the confidence scoring of viral contigs) — https://github.com/AnantharamanLab/VIBRANT
- **ViroProfiler** (Containerized pipeline that orchestrates the detection tools and applies the scoring system to classify viral contigs) — https://github.com/deng-lab/viroprofiler
- **CheckV** (Assesses viral genome quality class, contributing a criterion to the multi-tool viral contig classification) — https://bitbucket.org/berkeleylab/checkv

## Evaluation signals

- Every contig in the input contig library appears exactly once in viral_contig_confidence_table.tsv.
- Each contig carries a valid 'high confident' or 'low confident' label (>= 2 criteria vs. exactly 1).
- The criteria list per contig is consistent with the per-tool outputs, verified via field_present and row_count_equals checks.
- DeepVirFinder calls strictly follow the threshold: score > 0.9 and p < 0.01; borderline rows (e.g., score = 0.9 exactly) must be excluded.
- Contig identifiers are harmonized across all five tool outputs with no unmatched IDs.

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; use on total metagenomes with heavy environmental contamination increases runtime and lowers precision.
- Excessive environmental contaminations in metagenome sequences could increase the pipeline running time and result in lower precision; isolate viral fractions before sequencing for accurate estimation.
- A single DeepVirFinder hit (exactly one criterion) only yields a 'low confident' label; the threshold alone is not sufficient for high-confidence viral calls.

## Evidence

- [readme] DeepVirFinder predicts viral sequences using deep learning method.: [evidence span withheld — anchor status: external_doc]
