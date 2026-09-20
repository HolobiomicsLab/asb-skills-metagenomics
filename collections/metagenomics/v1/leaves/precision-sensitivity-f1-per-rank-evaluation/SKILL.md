---
name: precision-sensitivity-f1-per-rank-evaluation
description: Use when evaluating a viral metagenomic profiler on mock/simulated samples
  with known composition, and you need per-rank precision, sensitivity and F1 curves
  across abundance thresholds to compare tools.
when_to_use_negative:
- Samples have no ground-truth composition (real/unknown metagenomes) — precision/sensitivity
  cannot be computed
- Evaluating samples with excessive environmental contamination rather than isolated
  viral fractions — the paper notes contaminants can lower precision and are outside
  ViroProfiler's design scope
- Classifying viruses in metagenome sequences rather than samples with isolated viruses
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: ViroProfiler
  role: Viral metagenomics pipeline under test; provides assembly, binning, viral-contig
    identification and taxonomy assignment outputs to be scored against ground truth
  repo: https://github.com/deng-lab/viroprofiler
- name: Kraken2
  role: k-mer-based taxonomy assignment baseline run on the same FASTQ inputs in BrackenSTD
    and BrackenVPF configurations
- name: Bracken
  role: Produces abundance-corrected taxon tables from Kraken2 output for both reference
    database configurations
- name: Nextflow
  role: Workflow engine used to execute ViroProfiler reproducibly with a custom config
    (-c) and a docker/singularity profile
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:2fb4e739a2993881f5e307a1f58444e270dd78c95f8c36fb530aad79c5141f98
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# precision-sensitivity-f1-per-rank-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark viral taxonomy profilers (ViroProfiler vs Kraken2/Bracken configurations) by matching predicted taxa against mock-virome ground truth and computing precision (TP/(TP+FP)), sensitivity (TP/(TP+FN)) and F1 at each taxonomic rank and abundance threshold, as done in the ViroProfiler validation (Figure 2a).

## When to use

Use when evaluating a viral metagenomic profiler on mock/simulated samples with known composition, and you need per-rank precision, sensitivity and F1 curves across abundance thresholds to compare tools.

## When NOT to use

- Samples have no ground-truth composition (real/unknown metagenomes) — precision/sensitivity cannot be computed
- Evaluating samples with excessive environmental contamination rather than isolated viral fractions — the paper notes contaminants can lower precision and are outside ViroProfiler's design scope
- Classifying viruses in metagenome sequences rather than samples with isolated viruses

## Inputs

- 13 usable paired-end Illumina FASTQ pairs from 14 simulated mock virome samples (RefSeq v69-based)
- Known ground-truth taxonomic composition and abundances per mock sample
- Taxon abundance tables from ViroProfiler taxonomy module
- Bracken abundance-corrected taxon tables from BrackenSTD and BrackenVPF (Kraken2) configurations

## Outputs

- Per-tool, per-rank, per-threshold table of TP, FP, FN counts
- Precision, sensitivity and F1 values for each tool, taxonomic rank and abundance threshold
- Benchmark comparison table validating against Figure 2a values

## How to apply

Run each profiler end-to-end on the 13 usable paired-end FASTQ pairs from the 14 simulated Illumina mock virome samples (RefSeq v69-based): ViroProfiler via Nextflow with its default assembly (metaSPAdes), binning (vRhyme/phamb), viral-contig identification (VirSorter2/DeepVirFinder/VIBRANT) and taxonomy modules, plus Kraken2/Bracken in two reference configurations (BrackenSTD and BrackenVPF with the ViroProfiler-derived database). For each sample and each taxonomic rank, match predicted viral taxa and their abundances against the known ground-truth composition. Sweep abundance thresholds and, per threshold, count true positives, false positives and false negatives, then compute precision TP/(TP+FP), sensitivity TP/(TP+FN) and F1 (harmonic mean). Assemble the metrics into a single table ordered by tool, taxonomic rank and abundance threshold. Validate against the reported Figure 2a benchmark: ViroProfiler >95% precision and sensitivity at phylum level, BrackenVPF ~50% sensitivity.

## Related tools

- **ViroProfiler** (Viral metagenomics pipeline under test; provides assembly, binning, viral-contig identification and taxonomy assignment outputs to be scored against ground truth) — https://github.com/deng-lab/viroprofiler
- **Kraken2** (k-mer-based taxonomy assignment baseline run on the same FASTQ inputs in BrackenSTD and BrackenVPF configurations)
- **Bracken** (Produces abundance-corrected taxon tables from Kraken2 output for both reference database configurations)
- **Nextflow** (Workflow engine used to execute ViroProfiler reproducibly with a custom config (-c) and a docker/singularity profile)

## Examples

```
nextflow run deng-lab/viroprofiler --input 'samples.csv' -profile docker -c custom.config
```

## Evaluation signals

- Reproduced phylum-level ViroProfiler precision and sensitivity both exceed 95%, matching Figure 2a
- BrackenVPF sensitivity is approximately 50% at phylum level, consistent with the reported benchmark
- Per-threshold TP/(TP+FP) and TP/(TP+FN) values are finite and in [0,1] for every tool/rank cell
- F1 equals the harmonic mean of precision and sensitivity for each cell in the assembled benchmark table
- Any rank/threshold cell deviating materially from Figure 2a is flagged for review

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision
- It is recommended to isolate the viral fractions before sequencing for an accurate estimation of viruses in the samples
- One of the 14 simulated mock samples was unusable, so only 13 FASTQ pairs enter the benchmark

## Evidence
