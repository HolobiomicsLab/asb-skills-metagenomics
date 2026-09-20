---
name: cluster-count-verification
description: Use when a claimed clustering outcome (e.g. Metaclust's 424 million clusters)
  must be independently reproduced or validated from a published FASTA and reported
  Linclust parameters.
when_to_use_negative:
- The sequence set is already redundant-reduced or the input is a precomputed cluster
  mapping rather than a FASTA of representatives or raw sequences
- The clustering was performed with approximate-only prefiltering whose cluster members
  were not verified with the exact similarity measure, since pair verification still
  requires quadratic time and counts may not be reproducible
- Available compute cannot hold the multi-billion-sequence input; Linclust can exceed
  main memory via the MMseqs2 suite but a full reproduction still needs a single server-class
  machine
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Linear-time clustering algorithm used to reproduce the ≥50% sequence identity,
    90% shorter-sequence-coverage clustering and confirm the reported parameter configuration
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite integrating Linclust; supplies the mmseqs linclust command
    and workflow utilities used to run and verify clustering
  repo: https://github.com/soedinglab/mmseqs2
- name: Biopython SeqIO
  role: Alternative FASTA record counter for enumerating cluster representative headers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:5b95d503c00229a7472fb8530adcd022c4149f846447420e6cdef4b243924485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - Biopython SeqIO
  - Linclust
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cluster-count-verification

## Summary

Verifies large-scale protein sequence clustering results by counting cluster representatives in a FASTA set and reproducing Linclust/MMseqs2 clustering parameters, as done to confirm the Metaclust result of 424 million clusters from 1,595,926,152 input proteins clustered in 10 h.

## When to use

Use when a claimed clustering outcome (e.g. Metaclust's 424 million clusters) must be independently reproduced or validated from a published FASTA and reported Linclust parameters.

## When NOT to use

- The sequence set is already redundant-reduced or the input is a precomputed cluster mapping rather than a FASTA of representatives or raw sequences
- The clustering was performed with approximate-only prefiltering whose cluster members were not verified with the exact similarity measure, since pair verification still requires quadratic time and counts may not be reproducible
- Available compute cannot hold the multi-billion-sequence input; Linclust can exceed main memory via the MMseqs2 suite but a full reproduction still needs a single server-class machine

## Inputs

- FASTA file of cluster representatives (e.g. Metaclust from https://metaclust.mmseqs.org/)
- FASTA file of input protein sequences (1,595,926,152 metagenomic fragments predicted by Prodigal)
- Reported clustering parameters (--min-seq-id 0.5, --cov-mode 1, -c 0.9, --cluster-mode 2)

## Outputs

- Computed count of cluster representative (header) entries
- metaclust_cluster_verification.csv containing computed cluster count, reported values (1,595,926,152 input proteins, 424 million clusters, 10 h runtime), and relative deviation
- Optional Linclust cluster TSV output from a verification run on a subset

## How to apply

Download the clustering output set (e.g. Metaclust from https://metaclust.mmseqs.org/ or via metadownload.sh) as a FASTA file, then count representative (header) entries with a fast record counter (grep/awk or Biopython SeqIO). To confirm the clustering configuration, run Linclust (MMseqs2, commit 5e21868 in the paper) with the reported parameters --min-seq-id 0.5 --cov-mode 1 -c 0.9 --cluster-mode 2 (default E-value 10^-3) on a computationally feasible subset, noting coverage is of the shorter sequence. Compare the computed representative count against the reported value (424 million clusters from 1,595,926,152 input proteins; 10 h on a 2 × 14-core server) and report the relative deviation as the acceptance check. Greedy incremental clustering in Linclust always chooses the longest sequence as the cluster representative, so headers in the representative FASTA correspond one-to-one with clusters.

## Related tools

- **Linclust** (Linear-time clustering algorithm used to reproduce the ≥50% sequence identity, 90% shorter-sequence-coverage clustering and confirm the reported parameter configuration) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite integrating Linclust; supplies the mmseqs linclust command and workflow utilities used to run and verify clustering) — https://github.com/soedinglab/mmseqs2
- **Biopython SeqIO** (Alternative FASTA record counter for enumerating cluster representative headers)

## Examples

```
mmseqs linclust subset.fasta cluster_out tmp --min-seq-id 0.5 --cov-mode 1 -c 0.9 --cluster-mode 2; grep -c '>' metaclust_rep_seqs.fasta
```

## Evaluation signals

- The computed representative-sequence (header) count from the Metaclust FASTA is explicitly reported in the CSV
- Relative deviation between the computed cluster count and the reported 424 million clusters is stated and used as the acceptance check
- Verification run on a subset uses exactly the reported parameters: --min-seq-id 0.5 --cov-mode 1 -c 0.9 --cluster-mode 2 (default E-value 10^-3)
- Every cluster contains exactly one representative, so representative FASTA headers equal the number of clusters (one-to-one invariant)
- Input sequence count reconciles with the reported 1,595,926,152 proteins before clustering

## Limitations

- Linclust's k-mer prefilter is only approximate; falsely clustered pairs are not detected unless exact-distance verification is performed, which still requires quadratic time
- Full reproduction of the 1.59-billion-sequence Metaclust run required a 2 × 14-core server with a 10 h runtime; smaller machines must verify on subsets only
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting generalizability to other item types

## Evidence

- [abstract] Metaclust parameters: "We cluster 1.6 billion metagenomic sequence fragments in 10 h on a single server to 50% sequence identity, >1000 times faster than has been possible before."
- [methods] Linclust command line: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [discussion] Greedy representative choice: "We perform the clustering in step 5 of Fig. 1 with the greedy incremental clustering, because it always chooses the longest sequence as the cluster representative."
- [discussion] Prefilter approximation limitation: "If falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity."
- [methods] Metaclust download: "The Metaclust database is available as FASTA formatted file at https://metaclust.mmseqs.org/."
