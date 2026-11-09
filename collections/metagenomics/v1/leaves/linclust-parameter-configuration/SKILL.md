---
name: linclust-parameter-configuration
description: Use when a task requires clustering very large protein or nucleotide
  FASTA sets (billions of sequences) at a defined sequence identity and coverage threshold,
  or reproducing/re-verifying a reported Linclust/MMseqs2 clustering result.
when_to_use_negative:
- When exact clustering of every pair is required — Linclust's k-mer prefilter and
  hashing-based key selection are approximate; falsely clustered pairs are not verified
  beyond the exact-distance edge verification, which alone would be quadratic
- When the dataset is small enough that traditional tools like CD-HIT (run with -n
  5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5) suffice and quality differences matter
- When items cannot be assigned a set of m keys per item satisfying the shared-key
  condition, since Linclust's generalization requires such keys
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Primary linear-time clustering algorithm integrated into MMseqs2; run via
    'mmseqs linclust' with --cov-mode 1 -c 0.9 --min-seq-id flags
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite (GPLv3) that hosts the linclust command; provides search and
    clustering of huge protein and nucleotide sequence sets
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline comparator tool run at 90%, 70% and 50% sequence identity levels
    to evaluate Linclust cluster quality
- name: Prodigal
  role: Predicts the protein sequence fragments from metagenomic/metatranscriptomic
    data that form the 1.59-billion-sequence Metaclust input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:db1ac250864a819de31604a68cb0ba46c59bc74f637d360c25a876bff5fd6a1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - Prodigal
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

# linclust-parameter-configuration

## Summary

Configure MMseqs2's Linclust to cluster huge protein sequence sets in linear time using its k-mer prefilter pipeline; this skill encodes the paper's concrete parameters (e.g. --min-seq-id 0.5 --cov-mode 1 -c 0.9) and validation workflow for reproducing clustering results such as the 424-million-cluster Metaclust run.

## When to use

Use when a task requires clustering very large protein or nucleotide FASTA sets (billions of sequences) at a defined sequence identity and coverage threshold, or reproducing/re-verifying a reported Linclust/MMseqs2 clustering result.

## When NOT to use

- When exact clustering of every pair is required — Linclust's k-mer prefilter and hashing-based key selection are approximate; falsely clustered pairs are not verified beyond the exact-distance edge verification, which alone would be quadratic
- When the dataset is small enough that traditional tools like CD-HIT (run with -n 5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5) suffice and quality differences matter
- When items cannot be assigned a set of m keys per item satisfying the shared-key condition, since Linclust's generalization requires such keys

## Inputs

- Protein or nucleotide sequences in FASTA format (e.g. Metaclust protein set, UniProt 2016_03 release)
- Clustering parameters: minimum sequence identity (e.g. 0.5 or 0.9), coverage mode (--cov-mode 1), coverage threshold (-c 0.9), cluster mode

## Outputs

- Cluster representative sequences (FASTA of one representative per cluster)
- Cluster membership file mapping members to representatives
- Cluster count statistics for comparison against reported values (e.g. 424 million clusters)

## How to apply

Obtain the input protein sequences (e.g. the Metaclust FASTA set via the repository's metadownload.sh script or https://metaclust.mmseqs.org/) and run mmseqs linclust with coverage and identity flags: --cov-mode 1 -c 0.9 enforces 90% coverage of the shorter sequence, and --min-seq-id sets the identity cutoff (0.9 in the paper's benchmark; 0.5 for the Metaclust run), keeping the default E-value 10^-3. Optionally add --cluster-mode 2. Linclust works in five steps: it generates a k-mer table in a reduced 13-letter alphabet by selecting the m k-mers with the lowest hash values per sequence, sorts by k-mer index to find exact k-mer matches, applies a Hamming-distance pre-clustering and an ungapped alignment filter, then runs a gapped local alignment, and finally greedily clusters with the longest sequence as representative — giving O(N) runtime independent of the number of clusters K. For verification tasks on infeasibly large inputs, run Linclust on a computationally feasible subset to confirm the parameter configuration, count cluster representatives in the output FASTA, and compare against the reported value (e.g. 424 million clusters from 1,595,926,152 input proteins in 10 h on a 2 × 14-core server) stating the relative deviation as the acceptance check.

## Related tools

- **Linclust** (Primary linear-time clustering algorithm integrated into MMseqs2; run via 'mmseqs linclust' with --cov-mode 1 -c 0.9 --min-seq-id flags) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite (GPLv3) that hosts the linclust command; provides search and clustering of huge protein and nucleotide sequence sets) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline comparator tool run at 90%, 70% and 50% sequence identity levels to evaluate Linclust cluster quality)
- **Prodigal** (Predicts the protein sequence fragments from metagenomic/metatranscriptomic data that form the 1.59-billion-sequence Metaclust input)

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.5 --cluster-mode 2 metaclust.fasta result tmp
```

## Evaluation signals

- Computed number of cluster representatives in the output FASTA is explicitly reported and compared against the reported 424 million clusters, with relative deviation stated as the acceptance check
- Input sequence count matches the reported 1,595,926,152 proteins predicted by Prodigal
- Runtime is consistent with linear-time scaling — the reported run finished in 10 h on a 2 × 14-core server
- Clustering criteria reproduce the reported configuration: ≥50% sequence identity, 90% minimum coverage of the shorter sequence (--min-seq-id 0.5 --cov-mode 1 -c 0.9), default E-value 10^-3
- On UniProt 2016_03 benchmarks at 90%, 70% and 50% identity, cluster quality (functional consistency via GO annotations, local consistency via Pfam) is comparable to slower tools

## Limitations

- Linclust's prefilter is approximate: hashing and k-mer selection only approximately guarantee that sequences above the identity threshold share a selected k-mer, so very distant members of a true cluster may be missed
- Verification of all edges with the exact similarity measure would require quadratic time; Linclust trades this for linear O(mN) complexity including exact edge verification only within k-mer groups
- Traditional approaches are impracticably slow when both the number of sequences N and clusters K are in the tens of millions — but Linclust's quality is only benchmarked against such tools on smaller sets

## Evidence

- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [abstract] We cluster 1.6 billion metagenomic sequence fragments in 10 h on a single server to 50% sequence identity, >1000 times faster than has been possible before.: "We cluster 1.6 billion metagenomic sequence fragments in 10 h on a single server to 50% sequence identity, >1000 times faster than has been possible before."
- [methods] computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values: "computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values"
- [methods] The Metaclust database is available as FASTA formatted file at https://metaclust.mmseqs.org/.: "The Metaclust database is available as FASTA formatted file at https://metaclust.mmseqs.org/."
- [readme] MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets.: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
