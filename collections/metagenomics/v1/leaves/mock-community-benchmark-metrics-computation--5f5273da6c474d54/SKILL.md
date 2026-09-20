---
name: mock-community-benchmark-metrics-computation
description: Use when you have simulated/metagenomic mock community samples with known
  taxonomic composition and need to quantify a profiler's precision, sensitivity or
  F1 per taxonomic rank and abundance threshold.
when_to_use_negative:
- Input samples have no known ground-truth composition — precision/sensitivity cannot
  be computed without a reference
- Samples contain excessive environmental contamination rather than isolated viral
  fractions, which the article notes lowers precision and inflates running time
- The goal is functional annotation (DRAM-v, EggNOG, VOGDB) rather than taxonomic
  detection performance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ViroProfiler
  role: Candidate viral taxonomy profiler run end-to-end via Nextflow on mock samples
  repo: https://github.com/deng-lab/viroprofiler
- name: Kraken2
  role: Baseline taxonomic classifier whose abundance-corrected tables (via Bracken)
    are compared against ViroProfiler
- name: Bracken
  role: Abundance correction producing taxon tables in two database configurations
    (BrackenSTD, BrackenVPF)
- name: Nextflow
  role: Workflow engine used to execute ViroProfiler with containerized (docker/singularity)
    profiles
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:2af969609559c0e1782d21632b81ed9ec5444bf3981a9ad3d21f8d460578cbe0
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
  - Bracken
  - Kraken2
  - Nextflow
  - ViroProfiler
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mock-community-benchmark-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark viral taxonomic profilers (ViroProfiler vs Kraken2/Bracken) against the known ground truth of simulated mock virome samples by computing precision, sensitivity and F1 across taxonomic ranks and abundance thresholds.

## When to use

Use when you have simulated/metagenomic mock community samples with known taxonomic composition and need to quantify a profiler's precision, sensitivity or F1 per taxonomic rank and abundance threshold.

## When NOT to use

- Input samples have no known ground-truth composition — precision/sensitivity cannot be computed without a reference
- Samples contain excessive environmental contamination rather than isolated viral fractions, which the article notes lowers precision and inflates running time
- The goal is functional annotation (DRAM-v, EggNOG, VOGDB) rather than taxonomic detection performance

## Inputs

- 14 simulated Illumina paired-end mock virome FASTQ samples (RefSeq v69-based; 13 usable)
- Ground-truth taxonomic composition table per mock sample
- Taxon abundance tables from ViroProfiler, BrackenSTD and BrackenVPF

## Outputs

- Per-tool, per-rank, per-threshold table of TP, FP, FN counts
- Precision, sensitivity and F1 values per tool and taxonomic rank
- Benchmark table ordered by tool, taxonomic rank and abundance threshold

## How to apply

Run each tool on the same paired-end FASTQ inputs: ViroProfiler end-to-end via Nextflow (e.g. 'nextflow run deng-lab/viroprofiler' with a docker/singularity profile and a custom config passed with '-c'), and Kraken2/Bracken in two reference configurations (BrackenSTD standard database and BrackenVPF viral-focused database built from the ViroProfiler-derived references). Match predicted viral taxa and abundances against the known ground-truth composition of each mock sample at each taxonomic rank (e.g. phylum). Sweep abundance thresholds, and at each threshold tally true positives, false positives and false negatives per tool and rank, then compute precision (TP/(TP+FP)), sensitivity (TP/(TP+FN)) and their harmonic mean F1. Assemble metrics into a single table ordered by tool, taxonomic rank and abundance threshold, and validate against the published benchmark (ViroProfiler >95% precision and sensitivity at phylum level; BrackenVPF ~50% sensitivity, Figure 2a).

## Related tools

- **ViroProfiler** (Candidate viral taxonomy profiler run end-to-end via Nextflow on mock samples) — https://github.com/deng-lab/viroprofiler
- **Kraken2** (Baseline taxonomic classifier whose abundance-corrected tables (via Bracken) are compared against ViroProfiler)
- **Bracken** (Abundance correction producing taxon tables in two database configurations (BrackenSTD, BrackenVPF))
- **Nextflow** (Workflow engine used to execute ViroProfiler with containerized (docker/singularity) profiles)

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker -c viroprofiler.config --input 'mock_samples/*_{1,2}.fastq.gz'
```

## Evaluation signals

- Reproduced ViroProfiler metrics show >95% precision and sensitivity at phylum level, matching Figure 2a
- BrackenVPF sensitivity lands around 50% at phylum level, matching the published benchmark
- TP+FN at each rank equals the number of ground-truth taxa for that sample/rank
- All three tools are evaluated on the identical 13 usable mock FASTQ pairs and identical abundance thresholds
- Any rank/threshold cell deviating materially from Figure 2a is flagged for investigation

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision
- It is recommended to isolate the viral fractions before sequencing for an accurate estimation of viruses in the samples
- One of the 14 simulated mock samples was excluded as unusable, so results depend on sample filtering decisions

## Evidence

- [results] At the phylum level, ViroProfiler achieved >95% sensitivity and precision, whereas BrackenVPF (Kraken2/Bracken with the custom ViroProfiler-derived database) had only ~50% sensitivity: [evidence span withheld — anchor status: not_found]
