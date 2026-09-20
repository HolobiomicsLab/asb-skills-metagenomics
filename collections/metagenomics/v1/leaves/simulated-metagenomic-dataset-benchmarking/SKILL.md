---
name: simulated-metagenomic-dataset-benchmarking
description: Use when evaluating or reproducing quality-filtering stringency and chimera-retention
  of amplicon (16S/ITS) pipelines on simulated datasets with known ground truth, e.g.
  comparing UPARSE truncation filtering to MICCA expected-error (EE) filtering.
when_to_use_negative:
- Input is a real (non-simulated) microbiome dataset without ground-truth OTU composition
  or known chimera fraction — retention fractions cannot be validated against reported
  values
- The goal is OTU-count accuracy or taxonomy benchmarking rather than quality-filtering
  and chimera-retention behavior
- Reads are Illumina paired-end rather than the 454-style single-end reads these synthetic
  datasets emulate
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: UPARSE
  role: Pipeline whose truncation-based quality filtering, chimera detection and dereplication
    steps are benchmarked on the simulated datasets
- name: MICCA
  role: Comparison pipeline implementing flexible expected-error (EE) based filtering,
    used as the reference for read-retention efficiency
  repo: https://github.com/compmetagen/micca
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:3770118e22654fa21c476077e155f8396258b233e981239ed434a1ae1c1de0f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - UPARSE
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simulated-metagenomic-dataset-benchmarking

## Summary

Reproduces and verifies pipeline-level filtering and chimera-retention benchmarks (e.g., UPARSE vs MICCA) on synthetic 16S/ITS amplicon datasets with a realistic 454 error model and 20% chimeric sequences, where ground-truth read and chimera fractions are known.

## When to use

Use when evaluating or reproducing quality-filtering stringency and chimera-retention of amplicon (16S/ITS) pipelines on simulated datasets with known ground truth, e.g. comparing UPARSE truncation filtering to MICCA expected-error (EE) filtering.

## When NOT to use

- Input is a real (non-simulated) microbiome dataset without ground-truth OTU composition or known chimera fraction — retention fractions cannot be validated against reported values
- The goal is OTU-count accuracy or taxonomy benchmarking rather than quality-filtering and chimera-retention behavior
- Reads are Illumina paired-end rather than the 454-style single-end reads these synthetic datasets emulate

## Inputs

- Simulated 16S-10 dataset (realistic 454 error model, 20% chimeric sequences) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- Simulated ITS-10 dataset (realistic 454 error model, 20% chimeric sequences) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- FASTQ quality scores for each read

## Outputs

- uparse_filtering_results.csv with per-dataset retained-read fraction and chimera-retention fraction
- Kept/total read counts per dataset after truncation-based quality filtering
- Count of chimeric sequences retained after chimera detection and dereplication

## How to apply

Download the simulated 16S-10 and ITS-10 datasets from the paper's deposit at ftp://ftp.fmach.it/metagenomics/micca/scirep/. Apply UPARSE's truncation-based quality filter, which truncates each read at the first base whose quality score falls below a specified threshold and enforces a minimum read length, and record kept/total read counts; UPARSE's stringency should keep only ~42.3% of reads on 16S-10 and ~66.8% on ITS-10. Run UPARSE's chimera detection and dereplication on the filtered reads and count chimeric reads retained, expecting the worst-case 15.6% (16S-10) and 15.5% (ITS-10). Contrast these with MICCA's more flexible sliding-window/maximum-expected-error (EE) filtering, which uses sequenced reads more efficiently. Write the four metrics (kept-read % and chimera-retention % per dataset) to uparse_filtering_results.csv and check them against the reported values.

## Related tools

- **UPARSE** (Pipeline whose truncation-based quality filtering, chimera detection and dereplication steps are benchmarked on the simulated datasets)
- **MICCA** (Comparison pipeline implementing flexible expected-error (EE) based filtering, used as the reference for read-retention efficiency) — https://github.com/compmetagen/micca

## Evaluation signals

- Retained-read fraction must equal 42.3% on 16S-10 and 66.8% on ITS-10 after UPARSE filtering
- Chimeric-read retention must equal the reported worst-case 15.6% (16S-10) and 15.5% (ITS-10)
- uparse_filtering_results.csv contains exactly four metrics (kept-read % and chimera % for each dataset) matching reported values
- MICCA's EE-based filtering should retain a higher read fraction than UPARSE on both datasets, consistent with the paper's claim that MICCA uses reads more efficiently

## Limitations

- Only simulated datasets with known OTU composition were used for validation, given the lack of official and well-studied benchmark datasets for metagenomic pipelines
- Results are specific to a realistic 454 error model; behavior on Illumina or Ion Torrent reads is not covered
- UPARSE's conservative filtering costs sequence data and its estimated OTU numbers converge to only ~64% of the real number on average

## Evidence

- [results] we have analysed the 16S-10 and ITS-10 datasets generated using a realistic error model for 454 reads and including 20% of chimera sequences: "we have analysed the 16S-10 and ITS-10 datasets generated using a realistic error model for 454 reads and including 20% of chimera sequences"
- [results] UPARSE kept only 42.3% and 66.8% of reads on the 16S-10 and ITS-10 datasets respectively: "UPARSE tends to discard a higher number of reads, keeping only 42.3% and 66.8% of the reads"
- [results] UPARSE retained the worst results in chimeric reads: 15.6% and 15.5% on 16S-10 and ITS-10 datasets: "UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets (15.6% and 15.5% respectively)"
- [results] UPARSE quality filtering truncates reads at first base with quality score below threshold: "based on truncating the reads at the first base position which has a quality score less than a specified threshold"
- [methods] Dataset download location for the synthetic datasets: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [readme] MICCA filters low quality sequences according to the maximum allowed expected error (EE) rate: "filters low quality sequences according to the maximum allowed **expected
  error (EE) rate** %"
