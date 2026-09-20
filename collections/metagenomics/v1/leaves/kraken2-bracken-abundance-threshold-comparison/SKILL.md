---
name: kraken2-bracken-abundance-threshold-comparison
description: Use when FASTQ pairs from a mock/metagenomic virome with known ground-truth
  composition must be classified by Kraken2/Bracken and compared to ViroProfiler calls
  to quantify precision and sensitivity across ranks and abundance thresholds.
when_to_use_negative:
- Samples are experimental metagenomes with environmental contamination and no ground-truth
  composition, so TP/FP/FN cannot be scored
- Samples contain un-isolated viral fractions — ViroProfiler is specifically designed
  for classifying viruses in samples with isolated viruses
- The goal is functional annotation or host prediction rather than taxonomic classification
  benchmarking
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline whose viral taxonomy calls are benchmarked
    against Kraken2/Bracken outputs
  repo: https://github.com/deng-lab/viroprofiler
- name: Kraken2
  role: k-mer taxonomic classifier run in two database configurations to generate
    predicted viral taxa for the mock samples
- name: Bracken
  role: Produces abundance-corrected taxon tables (BrackenSTD and BrackenVPF) used
    to compute precision/sensitivity at each abundance threshold
- name: Nextflow
  role: Workflow engine used to execute ViroProfiler end-to-end on each FASTQ pair
    via 'nextflow run deng-lab/viroprofiler'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:aabbff5e2f7193e00156c544c6e831ae78488c76aafaf7c7acfcd9c48c439a8e
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
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# kraken2-bracken-abundance-threshold-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark Kraken2/Bracken taxonomic classification against the ViroProfiler pipeline on the simulated 14-sample mock virome (RefSeq v69-based), sweeping abundance thresholds and computing precision, sensitivity and F1 per tool and taxonomic rank to reproduce the Figure 2a comparison.

## When to use

Use when FASTQ pairs from a mock/metagenomic virome with known ground-truth composition must be classified by Kraken2/Bracken and compared to ViroProfiler calls to quantify precision and sensitivity across ranks and abundance thresholds.

## When NOT to use

- Samples are experimental metagenomes with environmental contamination and no ground-truth composition, so TP/FP/FN cannot be scored
- Samples contain un-isolated viral fractions — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- The goal is functional annotation or host prediction rather than taxonomic classification benchmarking

## Inputs

- 13 usable Illumina paired-end mock virome FASTQ sample pairs (RefSeq v69-based simulation)
- Ground-truth taxonomic composition of each mock sample per rank
- Kraken2/Bracken reference databases: BrackenSTD and BrackenVPF (ViroProfiler-derived)
- Nextflow config file for ViroProfiler execution

## Outputs

- Abundance-corrected Bracken taxon tables for BrackenSTD and BrackenVPF configurations
- ViroProfiler predicted viral taxa and abundances per sample
- Benchmark table of precision, sensitivity and F1 per tool (ViroProfiler, BrackenSTD, BrackenVPF), taxonomic rank and abundance threshold
- Deviation flags for rank/threshold cells inconsistent with the Figure 2a reference values

## How to apply

Retrieve the 14 simulated Illumina paired-end mock virome samples and select the 13 usable FASTQ pairs as input. Run ViroProfiler end-to-end via Nextflow (docker or singularity profile, custom config with '-c') so assembly (metaSPAdes), binning (vRhyme/phamb), viral-contig identification (VirSorter2/DeepVirFinder/VIBRANT) and taxonomy modules use defaults. Run Kraken2 on the same samples under two reference configurations — BrackenSTD (standard database) and BrackenVPF (ViroProfiler-derived viral database) — to obtain abundance-corrected taxon tables. Match predicted viral taxa and abundances against each sample's known composition at each rank, sweep abundance thresholds, and compute TP/FP/FN per threshold, then precision (TP/(TP+FP)), sensitivity (TP/(TP+FN)) and harmonic-mean F1 for each tool, rank and threshold. Assemble metrics into one benchmark table ordered by tool, rank and threshold, validating against the reported expectation that ViroProfiler exceeds 95% precision and sensitivity at phylum level while BrackenVPF reaches only ~50% sensitivity.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline whose viral taxonomy calls are benchmarked against Kraken2/Bracken outputs) — https://github.com/deng-lab/viroprofiler
- **Kraken2** (k-mer taxonomic classifier run in two database configurations to generate predicted viral taxa for the mock samples)
- **Bracken** (Produces abundance-corrected taxon tables (BrackenSTD and BrackenVPF) used to compute precision/sensitivity at each abundance threshold)
- **Nextflow** (Workflow engine used to execute ViroProfiler end-to-end on each FASTQ pair via 'nextflow run deng-lab/viroprofiler')

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker -c custom.config --input 'samples/*_{1,2}.fastq.gz'
```

## Evaluation signals

- At phylum rank, ViroProfiler precision and sensitivity both exceed 95%, consistent with Figure 2a
- BrackenVPF sensitivity at phylum level falls around ~50%, while BrackenSTD is lower still
- Every (tool, rank, threshold) cell in the benchmark table contains finite precision = TP/(TP+FP), sensitivity = TP/(TP+FN) and F1 = harmonic mean values
- Predicted taxa for each of the 13 samples can be matched one-to-one against the known ground-truth mock composition with no unmatched tool output
- Threshold sweep produces monotonic precision/sensitivity trade-off curves per tool rather than irregular jumps, indicating consistent abundance filtering

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; excessive environmental contaminations in metagenome sequences could increase the running time of the pipeline and result in lower precision
- Mock-dataset reproducibility is limited: the discussion does not name the benchmark datasets, comparison tools, or published datasets used for validation
- Only 13 of the 14 simulated mock virome samples are usable, so one sample must be excluded from scoring

## Evidence

- [abstract] ViroProfiler is an in-silico workflow for shotgun viral metagenomics using containerization: [evidence span withheld — anchor status: not_found]
- [readme] vpfkit converts ViroProfiler output for R analysis: [evidence span withheld — anchor status: external_doc]
