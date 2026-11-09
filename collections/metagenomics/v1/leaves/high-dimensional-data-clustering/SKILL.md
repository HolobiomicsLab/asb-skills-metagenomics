---
name: high-dimensional-data-clustering
description: Use when binning assembled metagenomic contigs whose feature vectors
  are too high-dimensional for reliable Euclidean/Manhattan metrics — e.g. composition
  vectors built from multiple k-mer sizes — and hull-based assignment is preferred.
when_to_use_negative:
- When the number of bins cannot be anchored by single-copy marker gene seeds — CH-Bin
  relies on marker-gene analysis for both bin count (ScmSelectPercentile percentile
  of seed counts) and initial binning.
- When contigs are well separated in low-dimensional space and Euclidean/Manhattan-distance
  clustering suffices — the hull-distance QP machinery targets the high-dimensionality
  regime where those metrics become unreliable.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CH-Bin
  role: End-to-end implementation of the convex hull distance binning pipeline; clones
    and runs the whole workflow.
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: CoverM
  role: Computes sample-wise read coverage of assembled contigs that forms the M coverage
    dimensions of the feature matrix.
  repo: https://github.com/wwood/CoverM
- name: seq2vec
  role: Vectorizes contig sequences into k-mer frequency vectors supplying the O composition
    dimensions of the feature matrix.
  repo: https://github.com/anuradhawick/seq2vec
- name: quadprog / cvxopt
  role: Python QP solvers used to solve the convex hull distance minimization per
    contig (configured via AlgoDistanceMetric=convex).
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:724bfe39a92432f4e22d341ff9d27b2e7d20ea55c72f0843e4e2b02a17b8c59a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kdsuneraavinash/CH-Bin
  repo_url: https://github.com/kdsuneraavinash/CH-Bin
  tools:
  - CH-Bin
  - CoverM
  - Python
  - cvxopt
  - quadprog
  - quadprog / cvxopt
  - seq2vec
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# high-dimensional-data-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster high-dimensional metagenomic contig feature vectors (composition + coverage) by assigning each contig to the bin whose convex hull minimizes the convex hull distance, as implemented in CH-Bin.

## When to use

Use when binning assembled metagenomic contigs whose feature vectors are too high-dimensional for reliable Euclidean/Manhattan metrics — e.g. composition vectors built from multiple k-mer sizes — and hull-based assignment is preferred.

## When NOT to use

- When the number of bins cannot be anchored by single-copy marker gene seeds — CH-Bin relies on marker-gene analysis for both bin count (ScmSelectPercentile percentile of seed counts) and initial binning.
- When contigs are well separated in low-dimensional space and Euclidean/Manhattan-distance clustering suffices — the hull-distance QP machinery targets the high-dimensionality regime where those metrics become unreliable.

## Inputs

- Assembled metagenomic contigs (FASTA) filtered by ContigLengthFilterBp
- Sample-wise read coverage of contigs computed with CoverM
- Oligonucleotide composition vectors from seq2vec (multiple k-mer sizes, e.g. k=5 and k=6)
- Initial binning of seed contigs from single-copy marker gene analysis (marker.hmm via HMMER/FragGeneScan)

## Outputs

- binning-assignment.csv with two comma-separated columns (contig name, integer bin index)
- FASTA files of the bins written to the bins output directory

## How to apply

Represent each contig as an N x (O+M) matrix of concatenated oligonucleotide composition (multiple k values, e.g. k=5,6) and sample-wise coverage (CoverM) vectors, then obtain an initial binning of seed contigs from single-copy marker gene analysis (HMMER against marker.hmm, FragGeneScan) using the 75th percentile of marker contig counts as the bin count. For each non-seed contig y, retrieve its m nearest neighbours (AlgoNumNeighbors) from a precomputed pairwise distance matrix D, form the neighbour set P_m, and solve the convex QP min_w (-2 Q^T P w + w^T P^T P w) s.t. e^T w = 1, w >= 0 with quadprog or cvxopt; in the default configuration, first replace P^T.P by its nearest positive-definite matrix (Higham, 1988) so the problem becomes strictly convex. Assign y to the bin whose hull yields the minimum convex hull distance, keep seed assignments fixed, and iterate over non-seed contigs in random order until convergence or AlgoMaxIterations is reached.

## Related tools

- **CH-Bin** (End-to-end implementation of the convex hull distance binning pipeline; clones and runs the whole workflow.) — https://github.com/kdsuneraavinash/CH-Bin
- **CoverM** (Computes sample-wise read coverage of assembled contigs that forms the M coverage dimensions of the feature matrix.) — https://github.com/wwood/CoverM
- **seq2vec** (Vectorizes contig sequences into k-mer frequency vectors supplying the O composition dimensions of the feature matrix.) — https://github.com/anuradhawick/seq2vec
- **quadprog / cvxopt** (Python QP solvers used to solve the convex hull distance minimization per contig (configured via AlgoDistanceMetric=convex).)

## Examples

```
git clone https://github.com/kdsuneraavinash/CH-Bin && cd CH-Bin && python src/main.py --config config/default.ini
```

## Evaluation signals

- binning-assignment.csv exists with exactly two comma-separated columns (contig name, integer bin index).
- Every non-seed contig receives a valid bin assignment in 0..K-1, while seed contigs retain their Step-3 bin indices.
- The iterative assignment loop converges within AlgoMaxIterations.
- FASTA files for each bin are dumped into the bins output directory.
- On simulated benchmarks (e.g. Sim50), binning quality should approach the reported best result (F1 96.91% with k=5,6) as a sanity reference.

## Limitations

- The convex hull distance computation is approximate: P^T.P is replaced by its nearest positive-definite matrix (Higham 1988), which turns the convex QP into a strictly convex one rather than solving the original problem exactly.
- The number of bins and initial binning depend on single-copy marker gene availability; datasets with few marker genes may mis-estimate bin count via the 75th-percentile rule.
- CH-Bin only supports Linux, and installation requires manual builds of FragGeneScan, HMMER, and seq2vec plus libboost/cmake dependencies.
- Traditional Euclidean and Manhattan distance metrics become unreliable in high dimensional space — the motivating limitation this skill addresses, but also a caution when interpreting neighbour lookups used for P_m.

## Evidence

- [results] convex hull distance approximated via Higham + Goldfarb-Idnani QP: [evidence span withheld — anchor status: not_found]
- [abstract] motivation for hull-based clustering in high dimensions: "However, these tools use traditional Euclidean distance or Manhattan distance metrics which become unreliable in the high dimensional space."
- [abstract] binning = grouping contigs after assembly: "Metagenomics binning, which involves the grouping of contigs into bins that represent different taxonomic groups, is an important step of a typical metagenomic workflow followed after assembly."
- [methods] seed contigs come from marker gene analysis and fix bin count: "CH-Bin uses the 75 th percentile (3 rd quartile) of the contig counts of single-copy marker genes to determine the number of bins."
- [other] kernel/QP objective and constraints solved by quadprog/cvxopt: [evidence span withheld — anchor status: not_found]
- [readme] Linux-only dependency note: "Note that CH-Bin only supports linux."
