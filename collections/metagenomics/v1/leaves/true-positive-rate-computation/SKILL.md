---
name: true-positive-rate-computation
description: Use when validating an annotation-expansion method by removing known
  KO annotations from eggNOG (.emapper.annotations) results and scoring whether an
  HMM search correctly recovers the removed gene and annotation.
when_to_use_negative:
- Gene sequences have been truncated (e.g. 30% or 70% of original length deleted)
  — those are separate truncation benchmarks, not the untruncated TPR test
- You need Module completeness evaluation or GSMM gapfilling rather than benchmarking
  HMM-based KO recovery (use --skip_hmm / --skip_gsmm workflows instead)
- Input annotations are not in a supported KEGG KO format (eggnog / kaas / kofamkoala)
  with maintained file extensions such as .emapper.annotations
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0769
tools:
- name: KEMET
  role: Runs the HMM-based KO recovery search (--hmm_mode onebm) against mock-removed
    annotations and applies the default quality filter (threshold 0.43)
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Source of the KO annotations (.emapper.annotations) from which KOs are mock-removed
- name: Python
  role: Used programmatically to remove the three KO annotations per genome and to
    score recovery outcomes and compute true positive rates
  repo: https://github.com/Matteopaluh/KEMET
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:29ed835fdf1da99d17481ee0c75d0a8d52cd4b605e2d42d8b0b40294ee8a789c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - KEMET
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# true-positive-rate-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark skill that quantifies how reliably a functional annotation-recovery tool (KEMET's --hmm_mode onebm HMM search) restores deliberately mock-removed KO annotations, expressed as true positive rates (e.g. 55% for MAGs, 67% for complete genomes).

## When to use

Use when validating an annotation-expansion method by removing known KO annotations from eggNOG (.emapper.annotations) results and scoring whether an HMM search correctly recovers the removed gene and annotation.

## When NOT to use

- Gene sequences have been truncated (e.g. 30% or 70% of original length deleted) — those are separate truncation benchmarks, not the untruncated TPR test
- You need Module completeness evaluation or GSMM gapfilling rather than benchmarking HMM-based KO recovery (use --skip_hmm / --skip_gsmm workflows instead)
- Input annotations are not in a supported KEGG KO format (eggnog / kaas / kofamkoala) with maintained file extensions such as .emapper.annotations

## Inputs

- eggNOG-mapper v2 annotation files (.emapper.annotations) for complete genomes and MAGs
- Gene FASTA files for each genome (gene calls from Prodigal v2.6.3, default options)
- Mock-removal specification: three KO annotations to delete per genome

## Outputs

- Per-removal outcome records (recovered / not recovered per removed KO per genome)
- Aggregate true positive rates per genome class (e.g. 20/36 = 55% for MAGs, 8/12 = 67% for complete genomes)
- Summary table, e.g. ko_mock_removal_tpr.csv

## How to apply

Prepare eggNOG-mapper v2 annotation files (.emapper.annotations) for each genome (5 complete genomes and 12 MAGs in the reference study), then programmatically remove three KO annotations per genome from the input annotation set. Run KEMET for each genome with './kemet.py [FASTA_file] -a eggnog --hmm_mode onebm' so profile HMMs search the removed KOs against the unmodified gene sequences (no truncation). Apply the default HMM quality filter (--threshold_value 0.43) to classify hits as legit recovered KOs. Score each mock removal as a true positive when both the removed gene and its annotation are correctly recovered, then compute the true positive rate per genome class; the reference study recovered 20/36 (55%) for MAGs and 8/12 (67%) for complete genomes. Write per-removal outcomes and aggregate TPRs to an output CSV (e.g. ko_mock_removal_tpr.csv).

## Related tools

- **KEMET** (Runs the HMM-based KO recovery search (--hmm_mode onebm) against mock-removed annotations and applies the default quality filter (threshold 0.43)) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Source of the KO annotations (.emapper.annotations) from which KOs are mock-removed)
- **Python** (Used programmatically to remove the three KO annotations per genome and to score recovery outcomes and compute true positive rates) — https://github.com/Matteopaluh/KEMET

## Examples

```
./kemet.py [FASTA_file] -a eggnog --hmm_mode onebm
```

## Evaluation signals

- Recovery counts must equal the reported 20/36 (55%) true positive rate for MAGs and 8/12 (67%) for complete genomes
- Each removed KO is scored a true positive only when both the gene and the annotation are correctly recovered by the HMM search
- Hits classified as legit must pass the default HMM quality filter (--threshold_value 0.43)
- No sequence truncation is applied in this test — truncated-sequence results belong to a separate benchmark
- Output CSV contains one row per mock removal plus aggregate TPR columns (per-removal outcomes and rates)

## Limitations

- True positive rates are below 100% (55% for MAGs, 67% for complete genomes), so a substantial fraction of mock removals is not recovered
- MAG assembly gaps can cause recovery failures, consistent with MAG-based GSMMs being 'especially prone to reconstruction errors due to the gapped nature of metagenomic assemblies'
- The threshold_value 0.43 filter trades sensitivity for specificity — lowering or raising it changes which hits count as legit recoveries

## Evidence
