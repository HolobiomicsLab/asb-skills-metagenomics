---
name: false-negative-fraction-estimation
description: Use when you need to quantify the sensitivity (false-negative fraction)
  of a sequence-clustering run at a given identity threshold (e.g. 50%, 70%, 90%)
  by comparing it against a BLAST-based reference of homologous pairs.
when_to_use_negative:
- When the goal is speed or runtime benchmarking rather than clustering sensitivity
  — runtime is a separate evaluation in this workflow
- When falsely clustered pairs are costly and you require exact pairwise verification
  of all links rather than a sampled BLAST reference (exact verification is still
  quadratic)
- When the input is already a feature/cluster table rather than a sequence FASTA —
  this skill operates on sequence sets and cluster memberships
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0084
tools:
- name: Linclust
  role: Linear-time clustering tool under test; run with 'mmseqs linclust --cov-mode
    1 -c 0.9 --min-seq-id 0.5' to generate the clusterings whose false negatives are
    estimated
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Cascaded cluster workflow ('mmseqs cluster --cov-mode 1 -c 0.9 --min-seq-id
    0.5') used as a higher-sensitivity comparison baseline
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Comparison clustering tool run with '-T 16 -M 0 -n 3 -c 0.5' at the 50% identity
    level
- name: UCLUST 7.0.1090
  role: Comparison clustering tool run with '--id 0.5' at the 50% identity level
- name: BLAST
  role: All-against-all search of 1000 sampled representatives to build the reference
    set of threshold-satisfying pairs with a 90% minimum-coverage filter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:3e1624c3bec15212a990557ec32e07062f1d3abe2d490b0ce46b6e417159c824
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - BLAST
  - CD-HIT
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  - UCLUST 7.0.1090
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# false-negative-fraction-estimation

## Summary

Estimates how many clustering-threshold-satisfying sequence pairs a clustering tool misses by BLAST-searching 1000 randomly sampled cluster representatives and computing the false-negative fraction against a BLAST reference set. Used in the Linclust paper (Steinegger & Söding 2018) to compare Linclust, MMseqs2, CD-HIT and UCLUST at 90/70/50% identity.

## When to use

Use when you need to quantify the sensitivity (false-negative fraction) of a sequence-clustering run at a given identity threshold (e.g. 50%, 70%, 90%) by comparing it against a BLAST-based reference of homologous pairs.

## When NOT to use

- When the goal is speed or runtime benchmarking rather than clustering sensitivity — runtime is a separate evaluation in this workflow
- When falsely clustered pairs are costly and you require exact pairwise verification of all links rather than a sampled BLAST reference (exact verification is still quadratic)
- When the input is already a feature/cluster table rather than a sequence FASTA — this skill operates on sequence sets and cluster memberships

## Inputs

- Large protein FASTA sequence set (e.g. UniProt 2016_03, 123 million test sequences)
- Clusterings of that set produced by each tool at a fixed identity threshold (e.g. --min-seq-id 0.5)
- Random sample of 1000 representative sequences per tool (FASTA)
- BLAST all-against-all search results among sampled representatives

## Outputs

- benchmark_fnr_results.csv with one entry per benchmarked tool containing the false-negative fraction and cumulative sequence-identity distribution
- Reference set of true homologous pairs defined by the 90% minimum-coverage BLAST criterion

## How to apply

Cluster the test set (e.g. the 123-million-sequence UniProt-derived set) with each tool using a common criterion of minimum sequence identity plus 90% coverage (cov-mode 1, -c 0.9; CD-HIT via '-n 3 -c 0.5' at the 50% level; UCLUST via '--id 0.5'). From each tool's clustering, randomly sample 1000 representative sequences and run all-against-all BLAST among them. Filter BLAST hits by the 90% minimum-coverage criterion (aligned residues divided by the length of the shorter sequence, equivalent to cov-mode 1) to define the reference set of true homologous pairs. For each tool, compute the cumulative sequence-identity distribution over sampled representative pairs and the false-negative fraction — reference pairs missed by the tool's clustering — writing both per-tool metrics to a results CSV (benchmark_fnr_results.csv). Rationale: exhaustive pairwise comparison is quadratic and infeasible at this scale, so a BLAST search of a random sample of representatives serves as an affordable, unbiased approximation of the true clustering threshold-satisfying pairs.

## Related tools

- **Linclust** (Linear-time clustering tool under test; run with 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5' to generate the clusterings whose false negatives are estimated) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Cascaded cluster workflow ('mmseqs cluster --cov-mode 1 -c 0.9 --min-seq-id 0.5') used as a higher-sensitivity comparison baseline) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Comparison clustering tool run with '-T 16 -M 0 -n 3 -c 0.5' at the 50% identity level)
- **UCLUST 7.0.1090** (Comparison clustering tool run with '--id 0.5' at the 50% identity level)
- **BLAST** (All-against-all search of 1000 sampled representatives to build the reference set of threshold-satisfying pairs with a 90% minimum-coverage filter)

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5
```

## Evaluation signals

- benchmark_fnr_results.csv exists and contains an entry for every benchmarked tool
- Every computed false-negative fraction lies between 0 and 1
- Expected values at 50% identity: ~2% (Linclust/MMseqs2), ~10% (UCLUST), ~16% (Linclust-m80), ~28% (Linclust)
- Reference pairs are defined only by BLAST hits passing the 90% minimum-coverage criterion (aligned residues / length of the shorter sequence)
- Values reproducible against scripts and benchmark data at bitbucket.org/martin_steinegger/linclust-analysis

## Limitations

- The false-negative fraction is estimated from a random sample of 1000 representatives, not exhaustive all-against-all comparison, so the BLAST reference is itself approximate
- Linclust's prefilter to find k-mer matches is approximate and misses some true matches, which is exactly what this metric quantifies (28% FNR for Linclust vs 2% for Linclust/MMseqs2 at 50% identity)
- If falsely clustered pairs are costly, pairwise links would need verification with the exact similarity measure, which still requires quadratic time complexity

## Evidence

- [methods] Common clustering criterion across tools: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [methods] Linclust command-line parameters: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] Reproducibility of benchmark and analysis results: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available at"
- [discussion] Rationale: exact verification is quadratic: "If falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity."
