---
name: cluster-statistics-computation
description: Use when you have ≥2 clusterings of the same sequence set (e.g., Linclust
  at m=20 vs m=80 and a UCLUST reference at 50% identity) and need summary statistics
  to judge which clustering recovered more sensitivity and at what runtime cost.
when_to_use_negative:
- Clusterings are not computed over the same underlying sequence set or at the same
  identity/coverage thresholds, making average-cluster-size comparisons meaningless
- The goal is clustering quality assessed by functional annotation consistency (GO/Pfam)
  rather than by cluster-size sensitivity and runtime statistics
- Only one clustering exists and no reference clustering is available for comparison
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MMseqs2
  role: Runs Linclust clusterings (default m=20 and --kmer-per-seq 80) whose cluster
    counts, average cluster sizes, and runtimes are compared
  repo: https://github.com/soedinglab/mmseqs2
- name: Linclust
  role: Linear-time clustering algorithm producing the candidate clusterings at 50%
    sequence identity whose statistics are computed
  repo: https://github.com/soedinglab/mmseqs2
- name: UCLUST
  role: Produces the reference clustering at --id 0.5 against which Linclust sensitivity
    recovery is measured
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:1c2c84a8f290d0a5ac76df24a4bf5985310666de8405626d0916e1abb2792ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cluster-statistics-computation

## Summary

Compute and compare cluster-level statistics (e.g., average cluster size, cluster counts, runtime ratios) across sequence clusterings produced at a fixed identity threshold, to quantify sensitivity differences between tools such as Linclust (default m=20 vs m=80 k-mers per sequence) and a reference clusterer like UCLUST.

## When to use

Use when you have ≥2 clusterings of the same sequence set (e.g., Linclust at m=20 vs m=80 and a UCLUST reference at 50% identity) and need summary statistics to judge which clustering recovered more sensitivity and at what runtime cost.

## When NOT to use

- Clusterings are not computed over the same underlying sequence set or at the same identity/coverage thresholds, making average-cluster-size comparisons meaningless
- The goal is clustering quality assessed by functional annotation consistency (GO/Pfam) rather than by cluster-size sensitivity and runtime statistics
- Only one clustering exists and no reference clustering is available for comparison

## Inputs

- FASTA-format protein sequence set (e.g., UniProt 2016_03 subset)
- Clustering result files from Linclust (m=20 default) with --min-seq-id 0.5, --cov-mode 1, -c 0.9
- Clustering result files from Linclust with --kmer-per-seq 80 and identical thresholds
- Reference clustering result from UCLUST with --id 0.5
- Per-run runtime measurements (wall-clock times)

## Outputs

- Summary table (e.g., linclust_m80_comparison.csv) with average cluster sizes, runtimes, sensitivity recovered relative to UCLUST, and the m=80 runtime overhead ratio
- Per-clustering average cluster size statistics
- Runtime ratio (Linclust m=80 vs default m=20)

## How to apply

Cluster the same input (e.g., a UniProt 2016_03 subset) with each tool using identical thresholds — --min-seq-id 0.5, --cov-mode 1, -c 0.9 for Linclust (MMseqs2), varying only --kmer-per-seq (m=20 default vs m=80), and --id 0.5 for UCLUST. Record each run's wall-clock runtime, then parse each clustering result to compute the average cluster size as total sequence count divided by number of clusters — larger average clusters at the same identity cutoff indicate more sequences merged, i.e., recovered sensitivity toward the reference clustering. Compute the runtime ratio of Linclust-m80 to default Linclust to quantify overhead (expected ~1.5–2x). Tabulate average cluster sizes, runtimes, and sensitivity recovered relative to the reference (consistent with the Supplementary Fig. S4 trend) into a CSV such as linclust_m80_comparison.csv. Validate by checking that the m=80 clustering shows increased average cluster size relative to default while runtime overhead remains moderate; this is grounded in Linclust's O(mN) linear-time complexity, which keeps the m=80 run tractable.

## Related tools

- **MMseqs2** (Runs Linclust clusterings (default m=20 and --kmer-per-seq 80) whose cluster counts, average cluster sizes, and runtimes are compared) — https://github.com/soedinglab/mmseqs2
- **Linclust** (Linear-time clustering algorithm producing the candidate clusterings at 50% sequence identity whose statistics are computed) — https://github.com/soedinglab/mmseqs2
- **UCLUST** (Produces the reference clustering at --id 0.5 against which Linclust sensitivity recovery is measured)

## Examples

```
mmseqs linclust uniport2016_03.fasta clusterRes --kmer-per-seq 80 --min-seq-id 0.5 --cov-mode 1 -c 0.9
```

## Evaluation signals

- Average cluster size of the Linclust m=80 clustering is larger than that of default m=20, indicating recovered sensitivity toward the UCLUST reference
- Runtime ratio of Linclust m=80 vs m=20 stays moderate (approximately 1.5–2x), consistent with O(mN) linear-time scaling
- All three clusterings (Linclust m=20, Linclust m=80, UCLUST) were run on the same input sequences with the same 50% sequence identity threshold
- Summary CSV (linclust_m80_comparison.csv) contains one row per clustering with non-empty average-cluster-size and runtime values
- Total sequences assigned across clusters sums to the input sequence count for each clustering

## Limitations

- Sensitivity recovery from increasing m is approximate; the k-mer selection prefilter means Linclust clustering is not exhaustive, and verification of edges with the exact similarity measure is included in O(mN) but the prefilter itself is only approximate
- Runtime overhead comparisons are hardware- and configuration-dependent and may not transfer across servers
- Average cluster size is a coarse sensitivity proxy; it does not measure cluster correctness (functional or local consistency requires GO/Pfam annotation checks)

## Evidence

- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated.: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [discussion] Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure.: "Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [readme] MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets.: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
