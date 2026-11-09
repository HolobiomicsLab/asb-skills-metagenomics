---
name: scaling-exponent-analysis-of-clustering-tools
description: Use when benchmarking whether a sequence-clustering tool's wall-clock
  runtime grows linearly or quadratically with input set size N, or when reproducing
  published power-law runtime exponents at a fixed sequence-identity threshold.
when_to_use_negative:
- The goal is cluster quality (functional or annotation consistency) rather than runtime
  scaling — the article evaluates quality separately via GO annotations and Pfam.
- The input is not a sequence set amenable to nested random subsampling, so the N-dependence
  of runtime cannot be isolated.
- Only a single input size is available; a power-law fit T = a*N^b requires runtimes
  at multiple N values.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Target clustering tool whose runtime scaling exponent (expected near 1) is
    measured via mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Suite hosting Linclust; the cluster variant (mmseqs cluster with the same
    parameters) is benchmarked separately as Linclust/MMseqs2
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Competing greedy clustering tool benchmarked with -T 16 -M 0 -n 5 -c 0.9 to
    estimate its runtime exponent
- name: UCLUST
  role: Competing greedy clustering tool benchmarked with --id 0.9 and 90% minimum
    coverage of the shorter sequence
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:925a65553dfbb999912154aa26f04e8a2e40b9e0c7cf87bb23abd75bea01e65d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# scaling-exponent-analysis-of-clustering-tools

## Summary

Empirically measures how clustering runtime scales with input set size N by timing UCLUST, CD-HIT, Linclust/MMseqs2, and Linclust on nested UniProt subsets and fitting a power law T = a*N^b in log-log space, revealing near-linear scaling for Linclust vs. near-quadratic scaling for competitors.

## When to use

Use when benchmarking whether a sequence-clustering tool's wall-clock runtime grows linearly or quadratically with input set size N, or when reproducing published power-law runtime exponents at a fixed sequence-identity threshold.

## When NOT to use

- The goal is cluster quality (functional or annotation consistency) rather than runtime scaling — the article evaluates quality separately via GO annotations and Pfam.
- The input is not a sequence set amenable to nested random subsampling, so the N-dependence of runtime cannot be isolated.
- Only a single input size is available; a power-law fit T = a*N^b requires runtimes at multiple N values.

## Inputs

- UniProt 2016_03 protein sequence sets (FASTA), sampled at 1/16, 1/8, 1/4, 1/2, and full size
- A UniProt version with reversed sequences (control set)
- Installed clustering tools: Linclust, Linclust/MMseqs2 (commit 5e21868), CD-HIT 4.6, UCLUST 7.0.1090
- A fixed similarity threshold (90% sequence identity, 90% minimum coverage of the shorter sequence)

## Outputs

- runtime_scaling.csv with one fitted power-law exponent b (and prefactor a) per tool
- Per-run wall-clock runtimes measured with the Linux time command

## How to apply

Prepare nested random subsets of the input corpus (here 1/16, 1/8, 1/4, and 1/2 of the UniProt 2016_03 release, plus a reversed-sequence version) so runtime can be measured at multiple N. Cluster each subset with each tool at a fixed identity and coverage threshold — Linclust and Linclust/MMseqs2 (commit 5e21868) with mmseqs linclust/cluster --cov-mode 1 -c 0.9 --min-seq-id 0.9, CD-HIT 4.6 with -T 16 -M 0 -n 5 -c 0.9, and UCLUST 7.0.1090 with --id 0.9 plus 90% minimum coverage of the shorter sequence to enforce global similarity. Measure wall-clock time per run with the Linux 'time' command. Fit T = a*N^b per tool by linear regression in log-log space using numpy/scipy, and write the exponent b and prefactor a to runtime_scaling.csv. The rationale is that the fitted exponent directly tests each tool's claimed complexity: Linclust is claimed to be O(mN) (linear), while greedy tools like CD-HIT and UCLUST are expected to scale almost quadratically since their runtime scales as N times the number of clusters K.

## Related tools

- **Linclust** (Target clustering tool whose runtime scaling exponent (expected near 1) is measured via mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Suite hosting Linclust; the cluster variant (mmseqs cluster with the same parameters) is benchmarked separately as Linclust/MMseqs2) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Competing greedy clustering tool benchmarked with -T 16 -M 0 -n 5 -c 0.9 to estimate its runtime exponent)
- **UCLUST** (Competing greedy clustering tool benchmarked with --id 0.9 and 90% minimum coverage of the shorter sequence)

## Examples

```
mmseqs linclust subsetDB subsetClu tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- runtime_scaling.csv exists with one exponent per tool (Linclust, Linclust/MMseqs2, UCLUST, CD-HIT).
- Fitted exponent for Linclust is near 1 (linear scaling, consistent with the O(mN) claim).
- Linclust/MMseqs2 exponent near 0.94–1.01, confirming linear behavior of both variants.
- UCLUST (~1.62) and CD-HIT (~2.75) exponents substantially larger, approaching quadratic behavior.
- Each tool was run at the identical threshold (90% identity, 90% coverage of the shorter sequence) across all subset sizes.

## Limitations

- Approximate prefilter-based algorithms (hashing, canopy preclustering, k-mer prefilter) trade sensitivity for speed; verifying falsely clustered pairs with exact similarity still requires quadratic time complexity.
- CD-HIT and UCLUST only accept the parameters given and enforce coverage externally, so their measured exponents depend on the chosen threshold and coverage enforcement.
- Exponents are fitted on nested UniProt subsets at 90% identity; other corpora or identity thresholds may shift the fitted values.

## Evidence

- [results] Fitted power-law exponents at 90% identity show UCLUST (N^1.62) and CD-HIT (N^2.75) scale very roughly quadratically, whereas Linclust/MMseqs2 (N^0.94) and Linclust (N^1.01) scale linearly with input set size.: [evidence span withheld — anchor status: not_found]
- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure.: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [abstract] We developed Linclust, the first clustering algorithm whose runtime scales as N, independent of K.: "We developed Linclust, the first clustering algorithm whose runtime scales as N, independent of K."
