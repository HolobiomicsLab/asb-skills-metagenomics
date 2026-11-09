---
name: reported-vs-computed-statistics-comparison
description: Use when a paper reports a concrete quantitative outcome (cluster count,
  runtime, input size) and the underlying dataset or pipeline is re-accessible, so
  the reported figure can be independently recomputed and compared.
when_to_use_negative:
- The reported statistic cannot be traced to a re-accessible artifact (dataset unavailable,
  no parameters given), leaving nothing to recompute
- The input is a full-scale benchmark claim (e.g., >1000x speedup) that depends on
  dedicated 2 x 14-core server hardware not available to the practitioner — verify
  only configuration, not runtime equivalence
- The claim is qualitative (e.g., 'clusters of comparable quality as other tools')
  rather than a countable figure
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: MMseqs2
  role: Software suite whose Linclust module reproduces the Metaclust clustering configuration;
    source code at github.com/soedinglab/mmseqs2
  repo: https://github.com/soedinglab/mmseqs2
- name: Linclust
  role: Linear-time clustering algorithm run with --min-seq-id 0.5 --cov-mode 1 -c
    0.9 --cluster-mode 2 to regenerate clusters whose count is compared to the reported
    424 million
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:8d1d26a23d211c4f392a8acb6490d61a7dc622c43c969f96759d1307d73c7b32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - Linclust
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# reported-vs-computed-statistics-comparison

## Summary

Recompute a published statistic from the underlying data or a feasible re-run and compare it numerically against the paper's reported value, stating relative deviation as the acceptance check. Applied here to the Metaclust clustering result (424 million clusters from 1,595,926,152 metagenomic proteins clustered in 10 h at ≥50% sequence identity).

## When to use

Use when a paper reports a concrete quantitative outcome (cluster count, runtime, input size) and the underlying dataset or pipeline is re-accessible, so the reported figure can be independently recomputed and compared.

## When NOT to use

- The reported statistic cannot be traced to a re-accessible artifact (dataset unavailable, no parameters given), leaving nothing to recompute
- The input is a full-scale benchmark claim (e.g., >1000x speedup) that depends on dedicated 2 x 14-core server hardware not available to the practitioner — verify only configuration, not runtime equivalence
- The claim is qualitative (e.g., 'clusters of comparable quality as other tools') rather than a countable figure

## Inputs

- Metaclust protein sequence set (FASTA), from metadownload.sh or https://metaclust.mmseqs.org/
- Published reported statistics from the article (e.g., 1,595,926,152 input proteins; 424 million clusters; 10 h runtime)
- Linclust/MMseqs2 clustering output (TSV of cluster representatives) when the pipeline is re-run on a feasible subset

## Outputs

- Computed cluster representative count (integer)
- metaclust_cluster_verification.csv listing computed vs. reported values and relative deviation

## How to apply

First obtain the primary artifact the statistic describes — e.g., download the Metaclust protein sequence set as FASTA via the repository's metadownload.sh script or https://metaclust.mmseqs.org/. Second, recompute the statistic directly on the artifact: count cluster representative FASTA header entries (each header = one cluster in Linclust/MMseqs2 output) with a fast line/record counter such as grep/awk or Biopython SeqIO. Third, optionally re-execute the reported pipeline on a computationally feasible subset to confirm the configuration matches the paper's parameters (Linclust with --min-seq-id 0.5 --cov-mode 1 -c 0.9 --cluster-mode 2, default E-value 10^-3). Fourth, compile computed values alongside reported values (1,595,926,152 input proteins, 424 million clusters, 10 h runtime on a 2 x 14-core server) into a CSV (e.g., metaclust_cluster_verification.csv), with the relative deviation explicitly stated as the acceptance criterion.

## Related tools

- **MMseqs2** (Software suite whose Linclust module reproduces the Metaclust clustering configuration; source code at github.com/soedinglab/mmseqs2) — https://github.com/soedinglab/mmseqs2
- **Linclust** (Linear-time clustering algorithm run with --min-seq-id 0.5 --cov-mode 1 -c 0.9 --cluster-mode 2 to regenerate clusters whose count is compared to the reported 424 million) — https://github.com/soedinglab/mmseqs2

## Examples

```
grep -c '>' metaclust.fasta; mmseqs linclust subset.fasta subsetDB tmp --min-seq-id 0.5 --cov-mode 1 -c 0.9 --cluster-mode 2
```

## Evaluation signals

- Computed representative-sequence count is explicitly recorded in the CSV (not omitted)
- Relative deviation between computed and reported 424 million clusters is stated and small (indicating the correct FASTA was downloaded and counted)
- Re-run subset clustering uses the exact reported parameters (--min-seq-id 0.5, --cov-mode 1, -c 0.9) rather than defaults
- Input sequence count reconciles with the reported 1,595,926,152 input proteins
- CSV schema contains paired computed/reported columns for every statistic (cluster count, input size, runtime)

## Limitations

- Full-scale recomputation of runtime (10 h on a 2 x 14-core server) is infeasible on typical hardware; only the clustering configuration and output counts can be verified on subsets
- Linclust-style linear-time clustering is approximate: 'If falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity.'
- Counting FASTA headers assumes each header in the released Metaclust FASTA corresponds to exactly one reported cluster

## Evidence

- [abstract] Metaclust clustering scale claim: "We cluster 1.6 billion metagenomic sequence fragments in 10 h on a single server to 50% sequence identity, >1000 times faster"
- [methods] Linclust command-line parameters used in the paper: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] Metaclust dataset availability: "The Metaclust database is available as FASTA formatted file at https://metaclust.mmseqs.org/."
- [methods] Benchmark reproduction materials: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available"
