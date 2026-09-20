---
name: cumulative-identity-distribution-analysis
description: Use when benchmarking sequence clustering tools (Linclust, MMseqs2, CD-HIT,
  UCLUST) on huge sets and you need to quantify clustering quality (false negatives)
  without exhaustively comparing all pairs.
when_to_use_negative:
- You lack clustering outputs from the tools under test — this skill evaluates clusterings,
  it does not produce them.
- The dataset is small enough to compute exhaustive all-against-all similarities directly,
  making the 1000-representative sampling unnecessary.
- Your quality criterion differs from sequence identity with 90% coverage (cov-mode
  1); the reference definition must match the clustering threshold being evaluated.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MMseqs2
  role: Provides the Linclust clustering being evaluated and defines the coverage
    convention (cov-mode 1) used to define true pairs
  repo: https://github.com/soedinglab/mmseqs2
- name: Linclust
  role: One of the benchmarked clusterers; run as 'mmseqs linclust --cov-mode 1 -c
    0.9 --min-seq-id 0.5' at the 50% identity level
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Benchmarked clusterer run with '-T 16 -M 0 -n 3 -c 0.5' at the 50% identity
    level
- name: UCLUST
  role: Benchmarked clusterer run with '--id 0.5' (version 7.0.1090) at the 50% identity
    level
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:6d6fb64967bc7b1a42d384f14eb3936c0add5d021afddd6c5013d445ef0fe06d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# cumulative-identity-distribution-analysis

## Summary

Estimates how many sequence pairs satisfying a clustering threshold (e.g., 50% identity with 90% coverage, cov-mode 1) each clustering tool misses, by BLAST-searching a random sample of 1000 representative sequences all-against-all to build a reference set of true homologous pairs, then computing cumulative sequence-identity distributions and false-negative fractions per tool.

## When to use

Use when benchmarking sequence clustering tools (Linclust, MMseqs2, CD-HIT, UCLUST) on huge sets and you need to quantify clustering quality (false negatives) without exhaustively comparing all pairs.

## When NOT to use

- You lack clustering outputs from the tools under test — this skill evaluates clusterings, it does not produce them.
- The dataset is small enough to compute exhaustive all-against-all similarities directly, making the 1000-representative sampling unnecessary.
- Your quality criterion differs from sequence identity with 90% coverage (cov-mode 1); the reference definition must match the clustering threshold being evaluated.

## Inputs

- FASTA-formatted large protein sequence set (e.g., UniProt 2016_03 test set, 123 million sequences)
- Per-tool clustering outputs (representative sequences and cluster members) from Linclust, MMseqs2, CD-HIT 4.6, UCLUST
- Clustering threshold definition: minimum sequence identity (e.g., 0.5) plus minimum coverage 0.9 in cov-mode 1

## Outputs

- benchmark_fnr_results.csv with one entry per benchmarked tool: false-negative fraction and cumulative sequence-identity distribution over sampled representative pairs
- BLAST-filtered reference set of true homologous pairs (hits passing the 90% minimum-coverage criterion)

## How to apply

Cluster the large sequence set (e.g., the 123-million-sequence UniProt-derived test set) with each tool under a fixed threshold — for Linclust use 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5', for CD-HIT 4.6 use '-T 16 -M 0 -n 3 -c 0.5', and for UCLUST 7.0.1090 use '--id 0.5'. Because exhaustive all-against-all comparison is quadratic and infeasible, randomly sample 1000 cluster representatives from each tool's output and run BLAST all-against-all among them. Filter BLAST hits by the 90% minimum-coverage criterion (aligned residues divided by the length of the shorter sequence, matching cov-mode 1) to define the reference set of pairs that genuinely satisfy the clustering threshold. For each tool, compute the cumulative sequence-identity distribution over sampled representative pairs and the false-negative fraction: reference pairs missed by that tool's clustering. This design mirrors the canopy/prefilter trade-off discussed in the paper: approximate prefilters can miss pairs, so missed pairs must be estimated against an exact reference.

## Related tools

- **MMseqs2** (Provides the Linclust clustering being evaluated and defines the coverage convention (cov-mode 1) used to define true pairs) — https://github.com/soedinglab/mmseqs2
- **Linclust** (One of the benchmarked clusterers; run as 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5' at the 50% identity level) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Benchmarked clusterer run with '-T 16 -M 0 -n 3 -c 0.5' at the 50% identity level)
- **UCLUST** (Benchmarked clusterer run with '--id 0.5' (version 7.0.1090) at the 50% identity level)

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5 testDB resultDB tmp
```

## Evaluation signals

- benchmark_fnr_results.csv exists and contains an entry for every benchmarked tool (Linclust/MMseqs2, UCLUST, Linclust-m80, Linclust, CD-HIT, MMseqs2 cascade).
- Every reported false-negative fraction lies in [0, 1].
- Expected magnitude at 50% identity: ~2% FNR for Linclust/MMseqs2, ~10% UCLUST, ~16% Linclust-m80, ~28% Linclust.
- The reference set of true pairs was derived only from BLAST hits passing 90% minimum coverage (aligned residues / length of shorter sequence), consistent with cov-mode 1.
- Results are reproducible against the deposited scripts and benchmark data at https://bitbucket.org/martin_steinegger/linclust-analysis.

## Limitations

- The false-negative rate is estimated from a random sample of 1000 representatives, not from exhaustive comparison, so it carries sampling uncertainty.
- Approximate prefilters (hashing, canopy preclustering, k-mer prefilter) are only approximate; if falsely clustered pairs are costly, pairwise links would need verification with the exact similarity measure, which requires quadratic time.
- Reference quality depends on BLAST's ability to recover pairs meeting the identity/coverage thresholds; pairs BLAST itself misses are excluded from the reference.

## Evidence

- [methods] Linclust command-line parameters: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] rationale for approximate clustering verification: "If falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity."
- [methods] benchmark reproducibility: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available at"
