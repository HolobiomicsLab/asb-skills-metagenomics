---
name: similarity-network-integration
description: Use when two or more similarity matrices cover the same set of entities
  (e.g., hosts) and must be combined into one fused similarity network to feed a downstream
  matrix factorization or prediction model.
when_to_use_negative:
- The entity set is a bipartite pair (viruses and hosts) rather than a single entity
  type with multiple similarity views — the virus network here uses only oligonucleotide
  frequency similarity and is not fused.
- Only one similarity matrix is available per entity set, leaving nothing for the
  diffusion process to integrate.
- The downstream model expects raw dissimilarity values (d2*) rather than fused similarity
  matrices in the [0, 1] range.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: NumPy
  role: Array operations for KNN local weighting, SNF diffusion iterations, and element-wise
    averaging of status matrices
- name: Jie Ren's oligonucleotide frequency dissimilarity tools
  role: Produce the d2* dissimilarity values between hosts/viruses that are transformed
    to similarity before fusion
- name: ILMF-VH
  role: Repository containing similarity_os.py and main.py used to calculate Gaussian
    interaction profile kernel similarity between hosts and the SNF integration feeding
    ILMF
  repo: https://github.com/liudan111/ILMF-VH
provenance:
  source_task_ids:
  - task_008
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:76e820fdb487abbfb56bd8ca9266fb5be031cc040441055fe523463617ed528b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liudan111/ILMF-VH
  tools:
  - ILMF-VH
  - ILMF-VH (github.com/liudan111/ILMF-VH)
  - Jie Ren's oligonucleotide frequency dissimilarity tools
  - NumPy
  - PyDTI (stephenliu0423/PyDTI)
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# similarity-network-integration

## Summary

Fuses multiple similarity networks describing the same entity set (here, host oligonucleotide-frequency similarity and Gaussian interaction profile kernel similarity) into a single host-host similarity matrix using similarity network fusion (SNF), as done in ILMF-VH before kernelized logistic matrix factorization.

## When to use

Use when two or more similarity matrices cover the same set of entities (e.g., hosts) and must be combined into one fused similarity network to feed a downstream matrix factorization or prediction model.

## When NOT to use

- The entity set is a bipartite pair (viruses and hosts) rather than a single entity type with multiple similarity views — the virus network here uses only oligonucleotide frequency similarity and is not fused.
- Only one similarity matrix is available per entity set, leaving nothing for the diffusion process to integrate.
- The downstream model expects raw dissimilarity values (d2*) rather than fused similarity matrices in the [0, 1] range.

## Inputs

- Host similarity matrix derived from oligonucleotide frequencies (d2* dissimilarity from Jie Ren's tools, transformed to similarity)
- Host similarity matrix derived from Gaussian interaction profile (GIP) kernels
- SNF parameters: neighborhood size K (=5) and number of similarity types m (=2)

## Outputs

- Fused host-host similarity matrix (element-wise average of diffused status matrices), values in [0, 1], written with host labels preserved to host_snf_similarity_matrix.csv

## How to apply

Load each host similarity input separately: one derived from oligonucleotide frequencies (after transforming Jie Ren d2* dissimilarity values to similarities) and one from Gaussian interaction profile (GIP) kernels. Apply K-nearest-neighbor local weighting to each matrix so only each host's K most similar neighbors contribute, matching standard SNF status-matrix construction. Run the SNF iterative diffusion process for m=2 iterations (m equals the number of similarity types, here 2), each network's status matrix being updated from the other's similarity structure at every iteration. Compute the final fused host-host similarity matrix as the element-wise average P = (P1 + P2)/2 of the two diffused status matrices, preserving host labels. In the ILMF-VH parameter sensitivity analysis, the SNF neighbor number K was swept and K=5 was found optimal, alongside k=100 (AUPR 0.6305), alpha=0.0625, beta=0.25, lambda=2, and gamma=0.25.

## Related tools

- **NumPy** (Array operations for KNN local weighting, SNF diffusion iterations, and element-wise averaging of status matrices)
- **Jie Ren's oligonucleotide frequency dissimilarity tools** (Produce the d2* dissimilarity values between hosts/viruses that are transformed to similarity before fusion)
- **ILMF-VH** (Repository containing similarity_os.py and main.py used to calculate Gaussian interaction profile kernel similarity between hosts and the SNF integration feeding ILMF) — https://github.com/liudan111/ILMF-VH

## Examples

```
python code/main.py  # computes GIP host similarity (similarity_os.py) then fuses ONF + GIP via SNF (K=5, m=2) into host_snf_similarity_matrix.csv
```

## Evaluation signals

- Output file host_snf_similarity_matrix.csv exists and is square with dimensions equal to the number of hosts.
- All fused similarity values lie in the [0, 1] range.
- Fused matrix structure matches the SNF-integrated host-host similarity network reported in the paper.
- Host labels are preserved in the output so rows/columns map back to the correct hosts.
- With K=5 fed into the full ILMF-VH pipeline, five-fold cross-validation reproduces the reported AUPR optimum of 0.6305 at k=100.

## Limitations

- The SNF neighbor number K affects fusion quality and must be tuned; in ILMF-VH it was swept and K=5 was optimal.
- The d2* outputs from Jie Ren's tools are dissimilarities and must be explicitly transformed to similarities before fusion.
- The SNF implementation described uses m=2 diffusion processes; adding more host similarity types requires adjusting m.

## Evidence

- [abstract] Host network constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion: [evidence span withheld — anchor status: not_found]
- [methods] The SNF mechanism fuses the two host similarity matrices by iteratively updating normalized weight matrices across m parallel diffusion processes; for this model there are two host similarity types, so m is set to 2.: [evidence span withheld — anchor status: not_found]
- [readme] Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity.: "Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] code/similarity_os.py and code/main.py are used to calculate Gaussian interaction profile kernel similarity between hosts.: "code/similarity_os.py and code/main.py are used to calculate Gaussian interaction profile kernel similarity between hosts."
