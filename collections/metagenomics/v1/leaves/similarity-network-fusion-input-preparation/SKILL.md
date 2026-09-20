---
name: similarity-network-fusion-input-preparation
description: 'Use when preparing host-host (or drug-target analogous) similarity inputs
  for SNF: you have a binary interaction adjacency matrix Y and need kernel similarity
  beyond sequence information.'
when_to_use_negative:
- The input is a continuous similarity/distance matrix already (e.g. oligonucleotide
  frequency similarity from Jie Ren's d2* tools) — GIP kernel construction would be
  redundant.
- No known virus-host associations are available, so interaction profiles cannot be
  defined.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0084
tools:
- name: PyDTI
  role: Python library supplying kernel/matrix-factorization routines (numpy-based)
    reused for GIP kernel computation and downstream prediction in ILMF-VH
  repo: https://github.com/stephenliu0423/PyDTI
- name: ILMF-VH code (similarity_os.py, main.py)
  role: Scripts used to calculate Gaussian interaction profile kernel similarity between
    hosts
  repo: https://github.com/liudan111/ILMF-VH
- name: numpy
  role: Numerical backend for column extraction from Y, squared-norm computation,
    and bandwidth normalization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:7f33e494840b286cbf83c9bbb4ea8454a898ff4a613d9146e79ad093cf13e746
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stephenliu0423/PyDTI
  tools:
  - ILMF-VH code (similarity_os.py, main.py)
  - PyDTI
  - numpy
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# similarity-network-fusion-input-preparation

## Summary

Compute the Gaussian interaction profile (GIP) kernel host-host similarity matrix from the binary virus-host adjacency matrix Y, producing one of the similarity inputs (alongside oligonucleotide frequency similarity) that are fused via similarity network fusion (SNF) in the ILMF-VH pipeline.

## When to use

Use when preparing host-host (or drug-target analogous) similarity inputs for SNF: you have a binary interaction adjacency matrix Y and need kernel similarity beyond sequence information.

## When NOT to use

- The input is a continuous similarity/distance matrix already (e.g. oligonucleotide frequency similarity from Jie Ren's d2* tools) — GIP kernel construction would be redundant.
- No known virus-host associations are available, so interaction profiles cannot be defined.

## Inputs

- Binary virus-host adjacency matrix Y (rows = viruses, columns = hosts), with N_h host columns
- Host interaction profiles (columns of Y), binary vectors of associations with each virus

## Outputs

- Symmetric host-by-host GIP kernel similarity matrix with entries in [0, 1] and unit diagonal (e.g. gip_host_similarity.csv)
- SNF-ready similarity input to be integrated with oligonucleotide frequency similarity of hosts

## How to apply

Load the binary virus-host adjacency matrix Y with rows indexing viruses and columns indexing hosts; each host's interaction profile IP(h_i) is the i-th column of Y, a binary vector of its associations with each virus. Compute pairwise host similarity with the Gaussian interaction profile kernel using the normalized bandwidth gamma_h = r'_h / (mean over k of ||IP(h_k)||^2), with r'_h simply set to 1. Assemble the pairwise scores into a symmetric host-by-host similarity matrix with unit diagonal. In the ILMF-VH repository this is done by code/similarity_os.py and code/main.py, and the resulting matrix is fed into SNF together with oligonucleotide frequency similarity to build the host network used by kernelized logistic matrix factorization.

## Related tools

- **PyDTI** (Python library supplying kernel/matrix-factorization routines (numpy-based) reused for GIP kernel computation and downstream prediction in ILMF-VH) — https://github.com/stephenliu0423/PyDTI
- **ILMF-VH code (similarity_os.py, main.py)** (Scripts used to calculate Gaussian interaction profile kernel similarity between hosts) — https://github.com/liudan111/ILMF-VH
- **numpy** (Numerical backend for column extraction from Y, squared-norm computation, and bandwidth normalization)

## Examples

```
python code/similarity_os.py --input Y.csv --output gip_host_similarity.csv
```

## Evaluation signals

- Output matrix is square with shape (number of hosts) x (number of hosts).
- Matrix is symmetric.
- All entries lie in [0, 1].
- All diagonal entries equal 1.
- Bandwidth uses the normalized formula with r'_h = 1, i.e. gamma_h = 1 / mean_k ||IP(h_k)||^2, matching the paper's setting.

## Limitations

- The kernel is defined only from known associations in Y; hosts with empty or near-identical interaction profiles receive degenerate similarities.
- The bandwidth hyperparameter r'_h is simply set to 1 rather than tuned, which may be suboptimal for other datasets.
- GIP similarity captures only association-profile overlap, not genomic or sequence-based host relatedness; it must be fused (via SNF) with oligonucleotide frequency similarity for the full host network.

## Evidence

- [methods] interaction profile IP(hi) of host hi is the i-th column of the adjacency matrix Y: [evidence span withheld — anchor status: not_found]
- [methods] bandwidth formula with r'_h = 1: [evidence span withheld — anchor status: not_found]
- [readme] README usage of scripts: "code/similarity_os.py and code/main.py are used to calculate Gaussian interaction profile kernel similarity between hosts."
- [abstract] SNF integration of host similarities: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
