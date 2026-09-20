---
name: multi-pipeline-comparison-evaluation
description: Use when comparing how competing amplicon-seq pipelines estimate OTU
  richness, especially when a synthetic dataset with a known number of true OTUs is
  available for validation.
when_to_use_negative:
- The dataset lacks a known ground truth number of OTUs, so accuracy fractions cannot
  be computed — use consistency/robustness comparisons instead
- The goal is a biological community analysis (e.g. genus profiling of HMP or environmental
  samples) rather than pipeline benchmarking
- Input is already an OTU table or feature table; this skill operates on raw reads
  through each pipeline end-to-end
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MICCA
  role: Primary pipeline under test; runs quality filtering and `otu` OTU picking
    backed by VSEARCH and swarm
  repo: https://github.com/compmetagen/micca
- name: MICCA-FAST
  role: Variant under test that replaces the dereplication step with an exact string
    matching algorithm
- name: QIIME
  role: Comparison pipeline; overestimates OTU counts with increasing depth
- name: mothur
  role: Comparison pipeline; performances qualitatively similar to QIIME
- name: UPARSE
  role: Comparison pipeline with stringent truncation-based quality filtering; converges
    to ~64% of true OTU count
- name: VSEARCH
  role: Third-party engine wrapped by micca for `filter` and `otu` commands
- name: swarm
  role: Clustering algorithm used in micca's `otu` command for de novo OTU picking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:51662c70468c37f0373cda861fca303ed545d32f6a70716815b1118b06cdcd19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - MICCA-FAST
  - QIIME
  - UPARSE
  - VSEARCH
  - mothur
  - swarm
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multi-pipeline-comparison-evaluation

## Summary

Benchmark the accuracy of amplicon pipelines (MICCA, MICCA-FAST, QIIME, mothur, UPARSE) in estimating the number of OTUs as a function of sequencing depth on a synthetic dataset with a known ground truth (16S-R: 500 OTUs, 100000 reads), reporting rarefied-depth OTU-count curves as fractions of the true count.

## When to use

Use when comparing how competing amplicon-seq pipelines estimate OTU richness, especially when a synthetic dataset with a known number of true OTUs is available for validation.

## When NOT to use

- The dataset lacks a known ground truth number of OTUs, so accuracy fractions cannot be computed — use consistency/robustness comparisons instead
- The goal is a biological community analysis (e.g. genus profiling of HMP or environmental samples) rather than pipeline benchmarking
- Input is already an OTU table or feature table; this skill operates on raw reads through each pipeline end-to-end

## Inputs

- 16S-R synthetic amplicon dataset (500 OTUs, 100000 reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- Rarefied subsets of the dataset at fixed depths increasing in steps of 1000 reads
- Pipeline installations: MICCA (>=1.3.0, pip or compmetagen/micca Docker image), MICCA-FAST, QIIME, mothur, UPARSE

## Outputs

- Table of estimated OTU counts per pipeline per rarefaction depth, as a fraction of the 500 true OTUs
- Rarefaction-style convergence curves per pipeline (e.g. MICCA ~77%, MICCA-FAST ~68%, UPARSE ~64% of true count)
- Per-pipeline quality-filtering statistics (reads retained/discarded) and chimeric-read rates

## How to apply

Download the 16S-R synthetic dataset (500 true OTUs, 100000 sequence reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/. Run each pipeline's full workflow on the complete data and on rarefied subsets: rarefy the sample to a fixed depth of increasing size at steps of 1000 reads, then run quality filtering (MICCA's expected-error-based filtering vs UPARSE's truncation at the first base below a quality threshold) followed by OTU picking. For MICCA use the `otu` command backed by VSEARCH and swarm; for MICCA-FAST substitute the dereplication step with exact string matching; where feasible, run QIIME, mothur and UPARSE on the identical rarefied subsets. For each pipeline and depth, divide the number of estimated OTUs by the 500 true OTUs and assemble a single comparison table. Accept the result if pipelines converge to stable fractions of the true count (MICCA ~77%, MICCA-FAST ~68%, UPARSE ~64%) while QIIME and mothur overestimate with increasing depth, and additionally compare read-retention and chimeric-read rates.

## Related tools

- **MICCA** (Primary pipeline under test; runs quality filtering and `otu` OTU picking backed by VSEARCH and swarm) — https://github.com/compmetagen/micca
- **MICCA-FAST** (Variant under test that replaces the dereplication step with an exact string matching algorithm)
- **QIIME** (Comparison pipeline; overestimates OTU counts with increasing depth)
- **mothur** (Comparison pipeline; performances qualitatively similar to QIIME)
- **UPARSE** (Comparison pipeline with stringent truncation-based quality filtering; converges to ~64% of true OTU count)
- **VSEARCH** (Third-party engine wrapped by micca for `filter` and `otu` commands)
- **swarm** (Clustering algorithm used in micca's `otu` command for de novo OTU picking)

## Examples

```
docker run -v $PWD:/data compmetagen/micca micca otu -i /data/filtered.fa -o /data/otus -t /data/tax -d 0.97
```

## Evaluation signals

- MICCA and MICCA-FAST rarefaction curves converge toward the true OTU count, reported on average at approximately 77% and 68% of the real number
- UPARSE curve converges to approximately 64% of the real OTU number on average
- QIIME and mothur curves continue to overestimate with increasing sequencing depth rather than converging
- MICCA reads-discarded counts are very similar to QIIME's, while UPARSE keeps only 42.3% and 66.8% of reads on 16S-10 and ITS-10 — read-retention stats should be tabulated per pipeline
- Every pipeline is evaluated on the identical rarefied subsets (same depths at 1000-read steps) so differences are attributable to the pipelines

## Limitations

- No official and well-studied benchmark datasets exist for metagenomic pipelines; the paper could only validate on simulated/synthetic data with known OTU composition (16S-R, 16S-10, ITS-10)
- QIIME and mothur non-convergence in rarefaction makes it difficult to merge samples sequenced to different depths and may lead to sequencing to excessive depth, increasing experimental costs
- Existing software allow many parameter combinations, making results from different experiments difficult to document and compare
- UPARSE's stringent truncation-based filtering, combined with a minimum read length requirement, causes the loss of a large fraction of sequence data (only 42.3% and 66.8% retained on 16S-10 and ITS-10), which must be considered when interpreting its OTU counts

## Evidence

- [results] The sample was rarefied to a fixed depth of increasing size at steps of 1000 reads: "The sample was rarefied to a fixed depth of increasing size at steps of 1000 reads"
- [results] Convergence fractions of MICCA/MICCA-FAST: "MICCA and MICCA-FAST achieve the best performances, converging to a number of estimated OTUs that are on average approximately 77% and 68% of the real number"
- [results] UPARSE convergence: "UPARSE is more conservative, converging to a number of estimated OTUs that is approximately 64% on average of the real number"
- [readme] Dataset availability from README: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [readme] micca OTU picking capabilities: "**multithread** de novo greedy, closed-reference, open-reference and swarm OTU
  picking protocols;"
