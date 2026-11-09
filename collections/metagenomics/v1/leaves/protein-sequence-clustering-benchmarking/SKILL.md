---
name: protein-sequence-clustering-benchmarking
description: Use when comparing the runtime scalability, speed, or sensitivity of
  protein sequence clustering tools on large FASTA sets, or when reproducing power-law
  runtime exponents and false-negative rates at fixed sequence-identity thresholds
  (e.g. 50% or 90%).
when_to_use_negative:
- The input is a small sequence set where any tool finishes in minutes — the linear-vs-quadratic
  scaling differences being benchmarked will not be measurable
- The goal is pairwise search or alignment speed comparison (e.g. BLAST vs MMseqs2
  search) rather than full-set redundancy clustering
- Exact, verified clustering is mandatory and false positives are unacceptable without
  quadratic-time verification, since approximate k-mer prefiltering trades exactness
  for speed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust (MMseqs2)
  role: Linear-time clustering tool under benchmark; runtime scales as O(mN) independent
    of cluster number K
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Provides the linclust and cascaded cluster workflows; also run as a slower
    cascaded baseline for speedup and FNR comparison
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool run with -T 16 -M 0 and word sizes -n 5/-n 4/-n 3
    at 90%/70%/50% identity
- name: UCLUST 7.0.1090
  role: Baseline clustering tool run with --id 0.9 / --id 0.5, also used as the sensitivity
    reference for Linclust-m80
- name: BLAST
  role: All-against-all search of 1000 sampled cluster representatives to define the
    true clusterable-pair reference set for false-negative estimation
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:836112d44adee3e505f3fe9f8789dad295260da268215fcf9379659384ead9c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - BLAST
  - CD-HIT
  - CD-HIT 4.6
  - Linclust (MMseqs2)
  - MMseqs2
  - UCLUST
  - UCLUST 7.0.1090
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# protein-sequence-clustering-benchmarking

## Summary

A benchmarking method for protein sequence clustering tools (Linclust/MMseqs2, MMseqs, CD-HIT, UCLUST) that measures runtime scaling, speedup factors, and clustering quality (false-negative rates, cluster counts) on huge FASTA sequence sets such as UniProt 2016_03 (~123M sequences with reversed sequences) and Metaclust (1.59B metagenomic fragments).

## When to use

Use when comparing the runtime scalability, speed, or sensitivity of protein sequence clustering tools on large FASTA sets, or when reproducing power-law runtime exponents and false-negative rates at fixed sequence-identity thresholds (e.g. 50% or 90%).

## When NOT to use

- The input is a small sequence set where any tool finishes in minutes — the linear-vs-quadratic scaling differences being benchmarked will not be measurable
- The goal is pairwise search or alignment speed comparison (e.g. BLAST vs MMseqs2 search) rather than full-set redundancy clustering
- Exact, verified clustering is mandatory and false positives are unacceptable without quadratic-time verification, since approximate k-mer prefiltering trades exactness for speed

## Inputs

- FASTA-formatted protein sequence set (e.g. UniProt 2016_03 release, optionally with reversed sequences, ~123M sequences)
- Random subsets of the sequence set (1/16, 1/8, 1/4, 1/2 of UniProt) for scaling experiments
- Clustering tool executables: mmseqs linclust / mmseqs cluster (commit 5e21868), CD-HIT 4.6, UCLUST 7.0.1090
- BLAST for the false-negative reference search on sampled representatives

## Outputs

- runtime_scaling.csv — fitted power-law exponent b (and prefactor a) per tool
- runtime_benchmark.tsv — per-tool runtimes and Linclust speedup factors at 50% and 90% identity
- benchmark_fnr_results.csv — per-tool false-negative fractions and cumulative sequence-identity distributions
- linclust_m80_comparison.csv — average cluster sizes, runtimes, and m=80 sensitivity recovery metrics
- Clustered representatives and cluster membership files per tool

## How to apply

Prepare the benchmark input by sampling fixed fractions (1/16 to 1/2) of the UniProt 2016_03 release plus a reversed-sequence version, or use the full ~123M-sequence set. Cluster each subset with each tool under matched global-similarity criteria: mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9 (or 0.5), CD-HIT 4.6 with -T 16 -M 0 and word size matched to identity (-n 5 -c 0.9, -n 3 -c 0.5), and UCLUST 7.0.1090 with --id 0.9/0.5. Time each run with the Linux 'time' command and fit a power law T = a*N^b by log-log linear regression (numpy/scipy) to obtain the scaling exponent b per tool. For quality assessment, randomly sample 1000 cluster representatives per tool, BLAST them all-against-all, filter hits by 90% minimum coverage of the shorter sequence (cov-mode 1), and compute the false-negative fraction of clusterable pairs missed. Sensitivity can be raised by rerunning Linclust with --kmer-per-seq 80 (m=80 instead of default m=20), which recovers UCLUST-comparable sensitivity at 1.5–2x runtime cost.

## Related tools

- **Linclust (MMseqs2)** (Linear-time clustering tool under benchmark; runtime scales as O(mN) independent of cluster number K) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Provides the linclust and cascaded cluster workflows; also run as a slower cascaded baseline for speedup and FNR comparison) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool run with -T 16 -M 0 and word sizes -n 5/-n 4/-n 3 at 90%/70%/50% identity)
- **UCLUST 7.0.1090** (Baseline clustering tool run with --id 0.9 / --id 0.5, also used as the sensitivity reference for Linclust-m80)
- **BLAST** (All-against-all search of 1000 sampled cluster representatives to define the true clusterable-pair reference set for false-negative estimation)

## Examples

```
mmseqs linclust input.fasta clustered --cov-mode 1 -c 0.9 --min-seq-id 0.5
```

## Evaluation signals

- runtime_scaling.csv exists with one fitted exponent per tool; Linclust exponent near 1 (reported N^0.94–N^1.01) while UCLUST (N^1.62) and CD-HIT (N^2.75) approach quadratic scaling
- runtime_benchmark.tsv reproduces order-of-magnitude speedups: ~2300x vs UCLUST and ~4600x vs CD-HIT at 50% identity; ~100x vs UCLUST and ~62x vs CD-HIT at 90% identity
- benchmark_fnr_results.csv contains an entry for every benchmarked tool with false-negative fractions in [0,1]; reference values are 2% (Linclust/MMseqs2), 10% (UCLUST), 16% (Linclust-m80), 28% (Linclust) at 50% identity
- Linclust-m80 shows increased average cluster size (sensitivity toward the UCLUST reference) with runtime overhead of only 1.5–2x over default Linclust
- Fitted parameters and clustering outputs should be consistent with scripts and benchmark data at https://bitbucket.org/martin_steinegger/linclust-analysis

## Limitations

- Linclust at default m=20 has higher false-negative rates (28% at 50% identity) than UCLUST or MMseqs2 cascaded clustering; raising sensitivity requires --kmer-per-seq 80 at increased runtime
- Approximate linear-time algorithms (hashing, canopy preclustering, k-mer prefilter) are only approximate; if falsely clustered pairs are costly, verification with the exact similarity measure still requires quadratic time
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting its generalizability
- Runtime scaling exponents are 'very rough' fits; results depend on hardware (e.g. 2 x 14-core server) and software versions (commit 5e21868, CD-HIT 4.6, UCLUST 7.0.1090)

## Evidence

- [results] Fitted power-law exponents at 90% identity show UCLUST (N^1.62) and CD-HIT (N^2.75) scale very roughly quadratically, whereas Linclust/MMseqs2 (N^0.94) and Linclust (N^1.01) scale linearly: [evidence span withheld — anchor status: not_found]
- [methods] CD-HIT 4.6 run with -T 16 -M 0 and word sizes matched to identity thresholds: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] Linclust linear time complexity includes verification of all edges using the exact distance measure: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [results] At 50% identity Linclust misses 2–28% of clusterable pairs depending on tool, measured via BLAST of 1000 sampled representatives with 90% coverage: [evidence span withheld — anchor status: not_found]
- [readme] MMseqs2 is a software suite to search and cluster huge protein and nucleotide sequence sets, free and open source in C++: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
