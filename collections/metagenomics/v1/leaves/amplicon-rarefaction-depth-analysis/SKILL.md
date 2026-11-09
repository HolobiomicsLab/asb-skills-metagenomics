---
name: amplicon-rarefaction-depth-analysis
description: Use when validating OTU-count accuracy of a 16S amplicon pipeline against
  a ground-truth synthetic dataset with known OTU composition, or when comparing de
  novo OTU-picking pipelines across sequencing depths.
when_to_use_negative:
- Input data is real (non-synthetic) amplicon data with no known true OTU number,
  so accuracy of OTU-count estimation cannot be scored
- The goal is taxonomic composition or alpha-diversity estimation rather than OTU-count
  accuracy benchmarking
- The input is already an OTU table or feature table, i.e. OTU picking has already
  been performed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MICCA
  role: 'Primary pipeline: quality filtering (expected-error based) and de novo OTU
    picking via the ''otu'' command backed by VSEARCH and swarm; installable via pip
    or compmetagen/micca Docker image (>=1.3.0)'
  repo: https://github.com/compmetagen/micca
- name: MICCA-FAST
  role: Variant of MICCA in which the dereplication step is replaced by an exact string
    matching algorithm, benchmarked at each rarefied depth
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Comparison pipeline run on the same rarefied subsets; used as a baseline that
    overestimates OTU counts
- name: mothur
  role: Comparison pipeline; qualitatively similar to QIIME in overestimating OTU
    number with depth
- name: UPARSE
  role: Comparison pipeline; more conservative, converging to approximately 64% of
    the real OTU number
- name: VSEARCH
  role: Backend used by the micca 'otu', 'filter', 'classify', 'mergepairs' and 'msa'
    commands
- name: swarm
  role: De novo clustering backend used by the micca 'otu' command
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:ba02669396ddef7efe1ba8e1b6e3fa985712d06304c8c131680682938a48ef4f
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# amplicon-rarefaction-depth-analysis

## Summary

A benchmarking protocol that compares how accurately amplicon pipelines (MICCA, MICCA-FAST, QIIME, mothur, UPARSE) estimate the number of OTUs as a function of sequencing depth on a synthetic 16S-R dataset with 500 true OTUs, by rarefying reads in steps of 1000 and plotting estimated/true OTU-count curves.

## When to use

Use when validating OTU-count accuracy of a 16S amplicon pipeline against a ground-truth synthetic dataset with known OTU composition, or when comparing de novo OTU-picking pipelines across sequencing depths.

## When NOT to use

- Input data is real (non-synthetic) amplicon data with no known true OTU number, so accuracy of OTU-count estimation cannot be scored
- The goal is taxonomic composition or alpha-diversity estimation rather than OTU-count accuracy benchmarking
- The input is already an OTU table or feature table, i.e. OTU picking has already been performed

## Inputs

- 16S-R synthetic amplicon dataset (500 OTUs, 100000 reads) in FASTQ format, from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- Rarefied FASTQ/Fasta subsets at fixed depths increasing in steps of 1000 reads
- Known ground-truth OTU number (500) for the synthetic dataset

## Outputs

- OTU tables (number of estimated OTUs) per pipeline and per rarefaction depth
- Rarefaction curves of estimated OTUs as a fraction of the 500 true OTUs
- Comparison table across MICCA, MICCA-FAST, QIIME, mothur and UPARSE

## How to apply

Download the 16S-R synthetic dataset (500 OTUs, 100000 sequence reads) from the paper's deposit at ftp://ftp.fmach.it/metagenomics/micca/scirep/. Run MICCA (>=1.3.0, via pip or the compmetagen/micca Docker image) quality filtering — based on the maximum allowed expected error (EE) rate — followed by the 'otu' de novo OTU-picking command backed by VSEARCH and swarm, on the full dataset and on subsets rarefied to a fixed depth of increasing size at steps of 1000 reads. Repeat with MICCA-FAST, which replaces the dereplication step with an exact string matching algorithm, and where feasible with QIIME, mothur and UPARSE on identical rarefied subsets. For each pipeline and depth, compute the number of estimated OTUs as a fraction of the 500 true OTUs and assemble a rarefaction table; convergence toward the true count (higher and flatter curves) indicates a more accurate pipeline, while continued overestimation with depth (QIIME, mothur) indicates inflated OTU counts.

## Related tools

- **MICCA** (Primary pipeline: quality filtering (expected-error based) and de novo OTU picking via the 'otu' command backed by VSEARCH and swarm; installable via pip or compmetagen/micca Docker image (>=1.3.0)) — https://github.com/compmetagen/micca
- **MICCA-FAST** (Variant of MICCA in which the dereplication step is replaced by an exact string matching algorithm, benchmarked at each rarefied depth) — https://github.com/compmetagen/micca
- **QIIME** (Comparison pipeline run on the same rarefied subsets; used as a baseline that overestimates OTU counts)
- **mothur** (Comparison pipeline; qualitatively similar to QIIME in overestimating OTU number with depth)
- **UPARSE** (Comparison pipeline; more conservative, converging to approximately 64% of the real OTU number)
- **VSEARCH** (Backend used by the micca 'otu', 'filter', 'classify', 'mergepairs' and 'msa' commands)
- **swarm** (De novo clustering backend used by the micca 'otu' command)

## Evaluation signals

- MICCA and MICCA-FAST rarefaction curves converge toward the true 500 OTU count, reported on average at approximately 77% and 68% of the real number respectively
- UPARSE curves converge to approximately 64% on average of the real number of OTUs
- QIIME and mothur rarefaction curves continue to increase (overestimate) with sequencing depth rather than converging
- Number of reads discarded by MICCA and QIIME quality filtering should be very similar on the same subsets
- Estimated OTU counts at each depth are reported as a fraction of 500 true OTUs, assembled in a single comparison table

## Limitations

- No official and well-studied benchmark datasets exist for metagenomic pipelines, so validation relies on simulated/synthetic data with known OTU composition
- Best pipelines still underestimate: even MICCA converges only to ~77% and MICCA-FAST to ~68% of the real OTU number on 16S-R
- QIIME and mothur non-convergence in rarefaction makes it difficult to merge samples sequenced to different depths and may lead to sequencing to excessive depth, increasing experimental costs
- UPARSE discards a large fraction of reads (42.3% on 16S-10, 66.8% on ITS-10) due to stringent quality filtering, which may affect comparability across pipelines

## Evidence

- [results] MICCA and MICCA-FAST estimated OTU numbers converged on average to approximately 77% and 68% of the real number respectively: "MICCA and MICCA-FAST achieve the best performances, converging to a number of estimated OTUs that are on average approximately 77% and 68% of the real number"
- [results] Rarefaction design: "The sample was rarefied to a fixed depth of increasing size at steps of 1000 reads"
- [results] 16S-R dataset definition: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [methods] Dataset deposit location: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [results] UPARSE conservative convergence: "UPARSE is more conservative, converging to a number of estimated OTUs that is approximately 64% on average of the real number"
- [results] MICCA-FAST dereplication variant: "MICCA-FAST, with the latter implementing an exact string matching dereplication algorithm"
- [readme] Micca OTU picking protocols: "multithread de novo greedy, closed-reference, open-reference and swarm OTU picking protocols;"
