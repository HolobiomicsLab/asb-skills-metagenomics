---
name: virus-host-adjacency-matrix-processing
description: Use when a binary virus-host association matrix Y is available and you
  need interaction-profile-based host-host (or virus-virus) kernel similarity to feed
  into similarity network fusion or kernelized matrix factorization.
when_to_use_negative:
- When only virus oligonucleotide frequency (d2*) dissimilarity data is available
  and no known virus-host associations exist — the GIP kernel requires the adjacency
  matrix's interaction profiles
- When the goal is virus-virus similarity, which in ILMF-VH is computed from oligonucleotide
  frequency (Jie Ren's d2* tools) rather than from interaction profiles
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PyDTI
  role: Python library (numpy/scipy-based) providing the interaction-profile kernel
    and matrix factorization machinery reused by ILMF-VH for virus-host prediction
  repo: https://github.com/stephenliu0423/PyDTI
- name: ILMF-VH code (similarity_os.py, main.py)
  role: Scripts that calculate the Gaussian interaction profile kernel similarity
    between hosts from the adjacency matrix
  repo: https://github.com/liudan111/ILMF-VH
- name: numpy
  role: Numerical library used to implement the kernel computation on the adjacency
    matrix in Python 2.7.9
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:ab74c9b1ce787021e30eb8eafbcced763bb1e69865a64ef7f98494585aae7550
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stephenliu0423/PyDTI
  tools:
  - ILMF-VH code (similarity_os.py, main.py)
  - PyDTI
  - numpy
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# virus-host-adjacency-matrix-processing

## Summary

Processes the binary virus-host adjacency matrix Y (rows = viruses, columns = hosts) to derive host-host similarity via the Gaussian interaction profile (GIP) kernel, where each host's interaction profile is a column of Y. This supplies the interaction-profile-based similarity that similarity network fusion combines with oligonucleotide frequency similarity in the ILMF-VH pipeline.

## When to use

Use when a binary virus-host association matrix Y is available and you need interaction-profile-based host-host (or virus-virus) kernel similarity to feed into similarity network fusion or kernelized matrix factorization.

## When NOT to use

- When only virus oligonucleotide frequency (d2*) dissimilarity data is available and no known virus-host associations exist — the GIP kernel requires the adjacency matrix's interaction profiles
- When the goal is virus-virus similarity, which in ILMF-VH is computed from oligonucleotide frequency (Jie Ren's d2* tools) rather than from interaction profiles

## Inputs

- Binary virus-host adjacency matrix Y (rows = viruses, columns = hosts; e.g. 820 viruses x 2699 hosts from dataset II, or 352 viruses x 71 hosts from dataset I)

## Outputs

- Symmetric host-by-host GIP kernel similarity matrix with entries in [0, 1] and unit diagonal (gip_host_similarity.csv), used as an input to similarity network fusion

## How to apply

Load the binary virus-host adjacency matrix Y with rows indexing viruses and columns indexing hosts (e.g. dataset II's 820 viruses x 2699 hosts from RefSeq). Extract each host's interaction profile IP(h_i) as the i-th column of Y, a binary vector of its associations with each virus. Compute pairwise host similarity with the Gaussian interaction profile kernel using the normalized bandwidth gamma_h = r'_h / (1/N_h * sum_k ||IP(h_k)||^2), with r'_h simply set to 1. Assemble the pairwise scores into a symmetric host-by-host similarity matrix with unit diagonal; in the ILMF-VH repository this is done by code/similarity_os.py and code/main.py (Python 2.7.9 with numpy), and the resulting matrix is passed to similarity network fusion together with oligonucleotide frequency similarity.

## Related tools

- **PyDTI** (Python library (numpy/scipy-based) providing the interaction-profile kernel and matrix factorization machinery reused by ILMF-VH for virus-host prediction) — https://github.com/stephenliu0423/PyDTI
- **ILMF-VH code (similarity_os.py, main.py)** (Scripts that calculate the Gaussian interaction profile kernel similarity between hosts from the adjacency matrix) — https://github.com/liudan111/ILMF-VH
- **numpy** (Numerical library used to implement the kernel computation on the adjacency matrix in Python 2.7.9)

## Examples

```
python code/similarity_os.py --input Y.csv --output gip_host_similarity.csv  # see code/main.py in liudan111/ILMF-VH
```

## Evaluation signals

- Output similarity matrix is square with dimensions equal to the number of hosts
- Matrix is symmetric and all entries lie in [0, 1]
- All diagonal entries equal 1 (self-similarity invariant)
- GIP kernel correctly derived: hosts sharing more virus associations in Y receive higher pairwise scores
- Matrix feeds SNF without dimension mismatch against the oligonucleotide-frequency host similarity matrix

## Limitations

- The GIP kernel only reflects known associations in Y — hosts absent from the adjacency (no sequenced host genome or no recorded interaction) get uninformative interaction profiles
- r'_h is simply set to 1, so the bandwidth is fixed by the normalization rather than tuned; results may be sensitive to this choice
- Code is implemented in Python 2.7.9 (per the README), which may require porting for modern environments

## Evidence

- [methods] The GIP kernel similarity between hosts is computed from interaction profiles, where the interaction profile IP(hi) of host hi is the i-th column of the adjacency matrix Y: [evidence span withheld — anchor status: not_found]
- [methods] bandwidth parameter formula with r'_h set to 1: [evidence span withheld — anchor status: not_found]
- [readme] README on GIP kernel computation scripts: "code/similarity_os.py and code/main.py are used to calculate Gaussian interaction profile kernel similarity between hosts."
- [abstract] Article abstract on host network construction: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
