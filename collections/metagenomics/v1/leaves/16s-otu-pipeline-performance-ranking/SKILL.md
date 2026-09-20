---
name: 16s-otu-pipeline-performance-ranking
description: Use when wall-clock running times of 16S OTU retrieval pipelines (CDSnake,
  DADA2, deblur, CD-HIT-OTU-MiSeq utilities) must be ranked per dataset from Table
  2-style runtime measurements to verify which pipeline is fastest or slowest.
when_to_use_negative:
- The goal is to compare OTU counts or annotation correctness rather than running
  times (use the OTU-count comparison findings instead)
- The input is already a computed feature/OTU table, so no pipeline execution or benchmarking
  is needed
- Datasets are not highly overlapping paired-end 2 × 250 bp reads, so the article's
  benchmark conditions do not apply
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CDSnake
  role: Snakemake pipeline under benchmark; wraps CD-HIT-OTU-MiSeq utilities for consecutive
    launch
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: QIIME2-based pipeline timed as a comparison baseline
- name: deblur
  role: QIIME2-based pipeline timed as a comparison baseline; fastest in every tested
    case
- name: CD-HIT-OTU-MiSeq
  role: Source utilities timed directly ('source CD-HIT') as a comparison baseline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:888ec26e6289f9bb60074fc26c2431e8b34eff674bc6f2bfe9de351cf88124df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-MiSeq
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# 16s-otu-pipeline-performance-ranking

## Summary

Benchmark and rank the wall-clock running times of four 16S rRNA OTU pipelines (CDSnake, DADA2, deblur, and the original CD-HIT-OTU-MiSeq utilities) on the Balanced, HMP, and Extreme mock community datasets plus their pooled 'All' set, so that pipeline selection can be justified by measured cost.

## When to use

Use when wall-clock running times of 16S OTU retrieval pipelines (CDSnake, DADA2, deblur, CD-HIT-OTU-MiSeq utilities) must be ranked per dataset from Table 2-style runtime measurements to verify which pipeline is fastest or slowest.

## When NOT to use

- The goal is to compare OTU counts or annotation correctness rather than running times (use the OTU-count comparison findings instead)
- The input is already a computed feature/OTU table, so no pipeline execution or benchmarking is needed
- Datasets are not highly overlapping paired-end 2 × 250 bp reads, so the article's benchmark conditions do not apply

## Inputs

- Reproduced Table 2 runtime measurements (seconds) for DADA2, deblur, source CD-HIT, and CDSnake
- Three mock community paired-end 2 × 250 bp datasets: Balanced, HMP, Extreme, plus the pooled 'All' set

## Outputs

- pipeline_runtime_ranking.csv containing per-dataset runtime rankings and flags indicating whether each claimed pattern holds

## How to apply

Load the reproduced Table 2 runtime measurements in seconds for DADA2, deblur, source CD-HIT, and CDSnake on the Balanced, HMP, Extreme mock datasets (2 × 250 bp paired-end reads with high overlap) and their pooled 'All' row. For each dataset row, sort the four pipelines by ascending runtime to compute each pipeline's speed rank, then check that deblur attains the minimum in every tested case (e.g. 1371.72 s on HMP). Verify the per-dataset patterns: CDSnake (2263.83 s) is slower than DADA2 (4614.81 s) is contradicted on HMP per the reported pattern where CDSnake is faster than DADA2, and the source CD-HIT utilities (2149.59 s on HMP) and CDSnake take substantially longer than the QIIME2-based pipelines on Extreme. Write the per-dataset ranking table with a flag per claimed pattern to pipeline_runtime_ranking.csv, and accept only if ranks match Table 2 values and each row contains exactly four ranked pipelines.

## Related tools

- **CDSnake** (Snakemake pipeline under benchmark; wraps CD-HIT-OTU-MiSeq utilities for consecutive launch) — github.com/ydkondratenko/cdsnake
- **DADA2** (QIIME2-based pipeline timed as a comparison baseline)
- **deblur** (QIIME2-based pipeline timed as a comparison baseline; fastest in every tested case)
- **CD-HIT-OTU-MiSeq** (Source utilities timed directly ('source CD-HIT') as a comparison baseline)

## Examples

```
cd <input_folder> && snakemake
```

## Evaluation signals

- deblur attains the minimum runtime in every tested case (e.g. 1371.72 s on HMP)
- Each dataset row in the output contains exactly four ranked pipelines
- The source CD-HIT-OTU-MiSeq utilities and CDSnake have the longest runtimes on the Extreme dataset
- Computed ranks match Table 2 values, including CDSnake faster than DADA2 on HMP and the pooled All row ordering
- Each claimed pattern is recorded as a pass/fail flag in pipeline_runtime_ranking.csv

## Limitations

- Hardware and execution environment for the time benchmarking are not specified in the article
- Software versions for DADA2, deblur, CD-HIT-OTU-MiSeq, or Snakemake are not stated
- No accessions or availability details are given for the three mock community datasets, limiting reproducibility

## Evidence

- [discussion] Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83: "Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83"
- [discussion] Time benchmarking of DADA2, deblur, source CH-HIT-OUT-Miseq utilities and CDSnake on three mock community datasets and set of these datasets (All row): "Time benchmarking of DADA2, deblur, source CH-HIT-OUT-Miseq utilities and CDSnake on three mock community datasets and set of these datasets (All row)"
- [abstract] Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking: "Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking - Balanced, HMP, and Extreme"
- [abstract] Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data.: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
