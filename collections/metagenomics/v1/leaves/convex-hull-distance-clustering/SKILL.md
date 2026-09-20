---
name: convex-hull-distance-clustering
description: Use when contigs are represented by high-dimensional feature vectors
  (concatenated oligonucleotide composition, e.g. k=5,6 k-mer frequencies, plus coverage)
  and you need clustering where Euclidean/Manhattan distances become unreliable.
when_to_use_negative:
- Input is raw reads or an already-produced binning/feature table rather than assembled
  contigs (binning follows assembly of reads into contigs)
- Low-dimensional feature representations where traditional Euclidean or Manhattan
  distance metrics remain reliable
- You lack single-copy marker gene information to seed initial bins, since CH-Bin
  relies on marker-gene-containing contigs to build seed bins and fix their assignments
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CH-Bin
  role: Implements the convex hull distance based iterative clustering (Cevikalp's
    algorithm) that assigns non-seed contigs to seed bins
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: quadprog
  role: Python QP solver used to solve the convex quadratic program min_w (-2 Q^T
    P w + w^T P^T P w) with e^T w = 1, w >= 0 (Goldfarb-Idnani)
- name: cvxopt
  role: Alternative configured QP solver for the convex hull distance quadratic program
- name: seq2vec
  role: Generates the k-mer frequency composition vectors (e.g. k=5,6) that form the
    O-dimensional part of each contig's feature vector
  repo: https://github.com/anuradhawick/seq2vec
- name: CoverM
  role: Computes sample-wise read coverage of assembled contigs, forming the M-dimensional
    coverage part of each feature vector
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:9cf8ace26c1b1a745aa8061ac67de6e04fc82e0a241d1ccb2348be74bd8561dc
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
  - FragGeneScan
  - HMMER3
  - Minimap2
  - Python
  - SAMTools
  - cvxopt
  - metaSPAdes
  - quadprog
  - seq2vec
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# convex-hull-distance-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns metagenomic contigs to bins by minimizing the convex hull distance between each contig's high-dimensional composition+coverage feature vector and the convex hull of each seed bin, as implemented in CH-Bin (Cevikalp's algorithm). It replaces unreliable Euclidean/Manhattan metrics in high-dimensional space with a convex quadratic program solved by quadprog or cvxopt.

## When to use

Use when contigs are represented by high-dimensional feature vectors (concatenated oligonucleotide composition, e.g. k=5,6 k-mer frequencies, plus coverage) and you need clustering where Euclidean/Manhattan distances become unreliable.

## When NOT to use

- Input is raw reads or an already-produced binning/feature table rather than assembled contigs (binning follows assembly of reads into contigs)
- Low-dimensional feature representations where traditional Euclidean or Manhattan distance metrics remain reliable
- You lack single-copy marker gene information to seed initial bins, since CH-Bin relies on marker-gene-containing contigs to build seed bins and fix their assignments

## Inputs

- N x (O+M) feature matrix of concatenated k-mer composition vectors (e.g. k=5,6 generated with seq2vec) and sample-wise read coverage values (computed with CoverM) for assembled contigs
- Initial seed binning: contigs containing single-copy marker genes (from FragGeneScan + HMMER3 against 107 marker HMMs) with fixed bin indices; non-seed contigs marked -1
- Configuration parameters: AlgoNumNeighbors (m), AlgoMaxIterations (T), AlgoDistanceMetric=convex, QP solver choice (quadprog/cvxopt)

## Outputs

- binning-assignment.csv with two comma-separated columns (contig name, integer bin index in 0..K-1)
- FASTA files of the final bins written to the bins output directory

## How to apply

Load the N x (O+M) feature matrix of concatenated composition and coverage vectors plus the initial seed binning from Step 3, where contigs containing single-copy marker genes are fixed and non-seed contigs are initialized to -1. For each non-seed contig y (visited in random order each iteration), retrieve its m nearest neighbors (AlgoNumNeighbors) from a precomputed pairwise distance matrix, form the set P_m, and solve min_w (-2 Q^T P w + w^T P^T P w) subject to e^T w = 1 and w >= 0 with the configured solver (quadprog or cvxopt; AlgoDistanceMetric = convex). In CH-Bin's default configuration, the nearest positive-definite matrix of P^T·P is computed via Higham's 1988 approach instead of the original P^T·P, reducing the convex problem to a strictly convex one that is then approximated via Goldfarb and Idnani's approach. Assign y to the bin whose convex hull yields the minimum hull distance, keeping seed-contig assignments fixed throughout. Repeat until the assignment vector converges or AlgoMaxIterations T is reached, then write contig-to-bin assignments to binning-assignment.csv and dump bin FASTA files.

## Related tools

- **CH-Bin** (Implements the convex hull distance based iterative clustering (Cevikalp's algorithm) that assigns non-seed contigs to seed bins) — https://github.com/kdsuneraavinash/CH-Bin
- **quadprog** (Python QP solver used to solve the convex quadratic program min_w (-2 Q^T P w + w^T P^T P w) with e^T w = 1, w >= 0 (Goldfarb-Idnani))
- **cvxopt** (Alternative configured QP solver for the convex hull distance quadratic program)
- **seq2vec** (Generates the k-mer frequency composition vectors (e.g. k=5,6) that form the O-dimensional part of each contig's feature vector) — https://github.com/anuradhawick/seq2vec
- **CoverM** (Computes sample-wise read coverage of assembled contigs, forming the M-dimensional coverage part of each feature vector) — https://github.com/wwood/CoverM

## Examples

```
w = quadprog.solve_qp(P.T@P, P.T@y, np.vstack([np.ones(m), np.eye(m)]), np.r_[1., np.zeros(m)], 1)[0]  # min -2(P^T y)^T w + w^T(P^T P)w s.t. e^T w=1, w>=0
```

## Evaluation signals

- binning-assignment.csv exists with two comma-separated columns (contig name, integer bin index)
- Every non-seed contig receives a valid bin assignment in 0..K-1; seed contigs retain their Step 3 marker-gene bin indices unchanged
- The iterative assignment converges within AlgoMaxIterations rather than hitting the iteration cap
- AMBER evaluation of the resulting bins on Sim50 with k=5,6 should reproduce average purity 95.28%, average completeness 98.60%, F1 score 96.91%, accuracy 93.10%, and 46 bins

## Limitations

- P^T·P may not be positive definite, so CH-Bin must approximate it via Higham's 1988 nearest positive-definite matrix approach before applying Goldfarb-Idnani; this makes the solution an approximation
- Seed-contig binning assignments are kept fixed throughout all iterations, so errors in the marker-gene-based initial binning propagate
- CH-Bin only supports Linux and requires external dependencies (FragGeneScan, HMMER, seq2vec) installed or path-configured in config/default.ini

## Evidence

- [methods] CH-Bin adopts the convex hull distance based clustering algorithm proposed by Cevikalp to bin metagenomic sequences.: [evidence span withheld — anchor status: not_found]
- [results] the nearest positive-definite matrix of P T .P is calculated using Higham′s approach (Higham, 1988), instead of using the original P T .P: [evidence span withheld — anchor status: not_found]
- [abstract] We further demonstrate that the convex hull distance based binning approach can be effectively utilized in binning such high dimensional data.: "We further demonstrate that the convex hull distance based binning approach can be effectively utilized in binning such high dimensional data."
- [abstract] However, these tools use traditional Euclidean distance or Manhattan distance metrics which become unreliable in the high dimensional space.: "However, these tools use traditional Euclidean distance or Manhattan distance metrics which become unreliable in the high dimensional space."
