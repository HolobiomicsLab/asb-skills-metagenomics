---
name: quadratic-programming-optimization
description: Use when computing convex hull distances for high-dimensional metagenomic
  contig feature vectors requires solving min_w (-2 Q^T P w + w^T P^T P w) s.t. e^T
  w = 1, w >= 0 via a QP solver.
when_to_use_negative:
- The problem is low-dimensional enough that Euclidean or Manhattan distance metrics
  are reliable — the article notes these become unreliable in high dimensional space,
  which is the motivation for hull distances
- You need only a single nearest-neighbor or centroid distance to a cluster; solving
  a QP per query-hull pair is substantially more expensive
- The Gram matrix P^T·P is already strictly positive-definite and a direct distance
  formula is available, making the Higham correction unnecessary
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CH-Bin
  role: Orchestrates the iterative convex hull distance binning loop and invokes the
    QP solver for each query contig / bin hull pair
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: quadprog
  role: Python QP solver implementing Goldfarb and Idnani's dual active-set method
    used to approximate the (strictly convex after Higham correction) convex hull
    distance program
- name: cvxopt
  role: Alternative configured QP solver for the same convex quadratic program when
    quadprog is not selected
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:ec6c078c76e91c4c7ba5a973a1ccd8d3cac36c84428dab82020bf0a346fe7262
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
  - Python
  - cvxopt
  - quadprog
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# quadratic-programming-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Solves the convex quadratic program that computes the convex hull distance of a query contig to a bin's hull in CH-Bin, using quadprog or cvxopt, with Higham's nearest positive-definite matrix correction when P^T·P is not positive-definite.

## When to use

Use when computing convex hull distances for high-dimensional metagenomic contig feature vectors requires solving min_w (-2 Q^T P w + w^T P^T P w) s.t. e^T w = 1, w >= 0 via a QP solver.

## When NOT to use

- The problem is low-dimensional enough that Euclidean or Manhattan distance metrics are reliable — the article notes these become unreliable in high dimensional space, which is the motivation for hull distances
- You need only a single nearest-neighbor or centroid distance to a cluster; solving a QP per query-hull pair is substantially more expensive
- The Gram matrix P^T·P is already strictly positive-definite and a direct distance formula is available, making the Higham correction unnecessary

## Inputs

- N x (O+M) feature matrix of concatenated oligonucleotide composition (multiple k values, e.g. k=5,6) and sample-wise read coverage vectors
- Initial binning assignment vector from Step 3 (seed contigs fixed, non-seed contigs initialized to -1)
- Precomputed pairwise distance matrix D between all contigs for nearest-neighbor lookups
- Configuration parameters: AlgoNumNeighbors, AlgoDistanceMetric=convex, AlgoMaxIterations, QP solver choice (quadprog or cvxopt)

## Outputs

- Per-contig convex hull distance values to each bin hull
- binning-assignment.csv with two comma-separated columns (contig name, integer bin index 0..K-1)
- FASTA files of the final bins dumped into the bins output directory

## How to apply

For each query contig y, retrieve its m nearest neighbors (AlgoNumNeighbors) to form the set P_m of hull points, then solve the convex quadratic program min_w (-2 Q^T P w + w^T P^T P w) subject to e^T w = 1 and w >= 0, where Q is the query point and P the neighbor matrix. In CH-Bin's default configuration, since the Gram matrix P^T·P over the m-neighbor set is not necessarily positive-definite, first compute its nearest positive-definite matrix using Higham's approach (1988); this converts the convex problem into a strictly convex one, which is then approximated via Goldfarb and Idnani's dual active-set algorithm as implemented in quadprog (or alternatively cvxopt). The minimized objective equals the convex hull distance of y to the hull of P_m. Solve one QP per (query contig, bin hull) pair each iteration, assign y to the bin with the minimum distance, and iterate until assignments converge or AlgoMaxIterations is reached, keeping seed-contig assignments fixed.

## Related tools

- **CH-Bin** (Orchestrates the iterative convex hull distance binning loop and invokes the QP solver for each query contig / bin hull pair) — https://github.com/kdsuneraavinash/CH-Bin
- **quadprog** (Python QP solver implementing Goldfarb and Idnani's dual active-set method used to approximate the (strictly convex after Higham correction) convex hull distance program)
- **cvxopt** (Alternative configured QP solver for the same convex quadratic program when quadprog is not selected)

## Examples

```
python -c "import numpy as np, quadprog; G=(P.T@P); Gp=nearest_pd(G); w=quadprog.solve_qp(Gp, -(-2*P.T@Q), np.vstack([np.ones(P.shape[0]), -np.eye(P.shape[0])]).T, np.array([1.]+[0.]*P.shape[0]), 1)[0]"
```

## Evaluation signals

- Every non-seed contig receives a valid integer bin assignment in 0..K-1 while seed contigs retain their Step 3 bin indices
- binning-assignment.csv exists with exactly two comma-separated columns (contig name, integer bin index)
- The iterative assignment loop converges within AlgoMaxIterations rather than hitting the iteration cap
- Each QP solution w satisfies the simplex constraints e^T w = 1 and w >= 0; when Higham's correction is applied, the corrected P^T·P passes a positive-definiteness check (e.g. Cholesky succeeds)
- On Sim50 with k=5,6 the pipeline reproduces an F1 score of ~96.91%, the best among tested tools

## Limitations

- P^T·P over a small neighbor set is not guaranteed positive-definite, so the Higham (1988) nearest-PSD correction is an approximation rather than the exact original P^T·P formulation
- Cost scales with the number of query contigs, bins, and QP solves per iteration; the README notes a trade-off where a change 'decreases binning slightly but drastically improves performance'
- CH-Bin only supports Linux, and performance depends on correctly locating external tools (FragGeneScan, HMMER, seq2vec) via config/default.ini

## Evidence

- [results] CH-Bin's default configuration approximates the convex hull distance by computing the nearest positive-definite matrix of P^T.P using Higham's approach: [evidence span withheld — anchor status: not_found]
- [other] QP formulation with simplex constraints: [evidence span withheld — anchor status: not_found]
- [other] Iterative assignment loop with fixed seeds: [evidence span withheld — anchor status: not_found]
- [methods] Tool applies Cevikalp's hull-based clustering to contigs: [evidence span withheld — anchor status: not_found]
