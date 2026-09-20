---
name: simulated-dataset-pipeline-validation
description: Use when benchmarking OTU-picking pipelines (MICCA, MICCA-FAST, QIIME,
  UPARSE, mothur) against a synthetic dataset whose true OTU count is known, to quantify
  estimation accuracy across sequencing depths.
when_to_use_negative:
- Input data has no ground truth (e.g. real HMP or environmental samples) — convergence
  to a true OTU count cannot be evaluated; use consistency or taxonomy-based checks
  instead
- The goal is downstream profiling accuracy (relative abundances, genus counts) rather
  than OTU-count estimation — use RSS of top-20 OTU relative abundances or RDP genus
  classification comparisons instead
- Datasets with substantially different read technology or OTU counts (e.g. 454-based
  16S-10/ITS-10 with 20% chimeras) where the chimeric-read and read-retention metrics,
  not OTU-count convergence, are the target
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MICCA
  role: 'Primary pipeline under test: quality filtering followed by de-novo OTU picking
    (otu command backed by VSEARCH and swarm) on full and rarefied subsets'
  repo: https://github.com/compmetagen/micca
- name: MICCA-FAST
  role: Variant of MICCA under test that replaces the dereplication step with an exact
    string matching algorithm
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Comparison pipeline run on identical rarefied subsets; known to overestimate
    OTU counts with increasing depth
- name: UPARSE
  role: Comparison pipeline with stringent truncation-based quality filtering; converges
    to approximately 64% of the real OTU number
- name: mothur
  role: Comparison pipeline; performances qualitatively similar to QIIME (overestimation)
- name: VSEARCH
  role: Backend used by MICCA's filter and otu commands during quality filtering and
    clustering
  repo: https://github.com/compmetagen/micca
- name: swarm
  role: De-novo clustering backend used by MICCA's otu command
  repo: https://github.com/compmetagen/micca
- name: sickle
  role: Sliding-window adaptive quality trimming tool listed in the article's context;
    alternative read-trimming utility
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:21b0b1d8969c54aad75e6890e3b53baa663f3e26a717a69dbd03c093a3be6a09
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
  - sickle
  - swarm
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# simulated-dataset-pipeline-validation

## Summary

Validates amplicon metagenomic pipelines by running them on synthetic datasets with known ground truth (e.g. the 16S-R dataset with 500 true OTUs and 100000 reads) and measuring how estimated OTU counts converge to the real number as sequencing depth increases via rarefaction.

## When to use

Use when benchmarking OTU-picking pipelines (MICCA, MICCA-FAST, QIIME, UPARSE, mothur) against a synthetic dataset whose true OTU count is known, to quantify estimation accuracy across sequencing depths.

## When NOT to use

- Input data has no ground truth (e.g. real HMP or environmental samples) — convergence to a true OTU count cannot be evaluated; use consistency or taxonomy-based checks instead
- The goal is downstream profiling accuracy (relative abundances, genus counts) rather than OTU-count estimation — use RSS of top-20 OTU relative abundances or RDP genus classification comparisons instead
- Datasets with substantially different read technology or OTU counts (e.g. 454-based 16S-10/ITS-10 with 20% chimeras) where the chimeric-read and read-retention metrics, not OTU-count convergence, are the target

## Inputs

- Synthetic amplicon FASTQ dataset with known OTU composition (16S-R: 500 OTUs, 100000 reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- Rarefied subsets of the dataset at fixed depths increasing in steps of 1000 reads
- Pipeline configurations for MICCA (>=1.3.0, pip or compmetagen/micca Docker image), MICCA-FAST, QIIME, mothur and UPARSE

## Outputs

- Per-pipeline table of estimated OTU counts at each rarefaction depth, expressed as a fraction of the 500 true OTUs
- Rarefaction (convergence) curves per pipeline showing estimated OTU count versus sequencing depth
- Average convergence percentages of estimated OTU counts relative to the real number of OTUs (e.g. MICCA ~77%, MICCA-FAST ~68%, UPARSE ~64%)

## How to apply

Download the synthetic ground-truth dataset (16S-R: 500 OTUs, 100000 sequence reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/. Run each pipeline end-to-end — quality filtering then OTU picking (for MICCA the `otu` command backed by VSEARCH and swarm; MICCA-FAST replaces dereplication with an exact string matching algorithm) — on the full dataset and on subsets rarefied to a fixed depth of increasing size at steps of 1000 reads. Assemble estimated OTU counts per pipeline and depth as a fraction of the 500 true OTUs. Judge convergence qualitatively: the paper reports MICCA and MICCA-FAST converging on average to approximately 77% and 68% of the real OTU number, while QIIME and mothur continue to overestimate with increasing depth and UPARSE converges to approximately 64%. This rarefaction-based convergence behaviour, not a single point estimate, is the core validation signal.

## Related tools

- **MICCA** (Primary pipeline under test: quality filtering followed by de-novo OTU picking (otu command backed by VSEARCH and swarm) on full and rarefied subsets) — https://github.com/compmetagen/micca
- **MICCA-FAST** (Variant of MICCA under test that replaces the dereplication step with an exact string matching algorithm) — https://github.com/compmetagen/micca
- **QIIME** (Comparison pipeline run on identical rarefied subsets; known to overestimate OTU counts with increasing depth)
- **UPARSE** (Comparison pipeline with stringent truncation-based quality filtering; converges to approximately 64% of the real OTU number)
- **mothur** (Comparison pipeline; performances qualitatively similar to QIIME (overestimation))
- **VSEARCH** (Backend used by MICCA's filter and otu commands during quality filtering and clustering) — https://github.com/compmetagen/micca
- **swarm** (De-novo clustering backend used by MICCA's otu command) — https://github.com/compmetagen/micca
- **sickle** (Sliding-window adaptive quality trimming tool listed in the article's context; alternative read-trimming utility) — https://github.com/najoshi/sickle

## Examples

```
docker run -v $PWD:/data compmetagen/micca micca otu -i /data/16s-r_filt.fa -o /data/otus -d /data/otus_table
```

## Evaluation signals

- MICCA and MICCA-FAST rarefaction curves converge toward the true OTU count, reported on average at approximately 77% and 68% of the 500 real OTUs
- QIIME and mothur curves continue to overestimate with increasing depth rather than plateauing, matching the qualitative behaviour reported in the paper
- UPARSE converges to a more conservative estimate of approximately 64% of the real OTU number on average
- Each rarefied subset differs by exactly the fixed step size (1000 reads), and all pipelines are run on the identical subsets
- Estimated OTU counts at each depth are reproducible as a fraction of 500 across pipeline re-runs

## Limitations

- Even the best pipelines do not reach the true OTU count: MICCA and MICCA-FAST converge only to approximately 77% and 68% of the real number respectively
- No official and well-studied benchmark datasets exist, so accuracy assessment generally relies on simulated data with known OTU composition
- Non-convergence of QIIME and mothur makes it difficult to merge samples sequenced to different depths and may lead to sequencing to excessive depth, increasing experimental costs
- UPARSE's stringent truncation-based quality filtering combined with a minimum read length requirement discards a large fraction of reads (only 42.3% and 66.8% kept on 16S-10 and ITS-10), which affects comparability on other datasets

## Evidence

- [results] MICCA and MICCA-FAST achieve the best performances, converging to a number of estimated OTUs that are on average approximately 77% and 68% of the real number: "MICCA and MICCA-FAST achieve the best performances, converging to a number of estimated OTUs that are on average approximately 77% and 68% of the real number"
- [results] rarefaction protocol: "The sample was rarefied to a fixed depth of increasing size at steps of 1000 reads"
- [results] 16S-R dataset description: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [results] MICCA-FAST dereplication: "MICCA-FAST, with the latter implementing an exact string matching dereplication algorithm"
- [methods] dataset download location: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [results] UPARSE convergence: "UPARSE is more conservative, converging to a number of estimated OTUs that is approximately 64% on average of the real number"
- [discussion] benchmark data gap: "Given the lack of official and well-studied datasets to test the analysis tools, it is generally difficult to assess the accuracy and reliability of a metagenomic pipeline"
