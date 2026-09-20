---
name: knn-local-weighting-of-networks
description: Use when multiple similarity matrices over the same node set (e.g., ONF-based
  and GIP kernel-based host similarities) must be locally sparsified to K neighbors
  before SNF-style fusion into a single similarity network.
when_to_use_negative:
- The similarity matrices come from a single information source and no network fusion
  is planned — KNN weighting before SNF is specific to integrating multiple host similarities
- The graph is already sparse or kNN-thresholded; re-applying KNN local weighting
  would discard legitimate edges
- The input is a dissimilarity matrix (e.g., raw Jie Ren d2* values) that has not
  yet been transformed to similarity
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: NumPy
  role: Implements the K-nearest-neighbor row-wise selection and construction of the
    locally weighted status matrices from the ONF and GIP similarity matrices
- name: ILMF-VH
  role: Repository containing the pipeline (similarity_os.py, main.py for GIP kernel
    similarity; nrlmf.py for downstream prediction) in which the KNN-weighted networks
    are fused via SNF
  repo: https://github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Python library (requires Numpy, scikit-learn, Scipy) providing the neighborhood-regularized
    logistic matrix factorization infrastructure whose conventions the KNN weighting/SNF
    pipeline follows
  repo: https://github.com/stephenliu0423/PyDTI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:b2368ba9b59e56847f50bbe290f7d2b8d39c2ba9117ad6a13d23223e2eace2d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liudan111/ILMF-VH
  tools:
  - ILMF-VH
  - NumPy
  - PyDTI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# knn-local-weighting-of-networks

## Summary

Applies K-nearest-neighbor local (KNN) weighting to each input similarity matrix — here the host oligonucleotide-frequency (ONF) similarity and the Gaussian interaction profile (GIP) kernel similarity — so only each node's K most similar neighbors contribute, as a prerequisite step before similarity network fusion (SNF) in ILMF-VH.

## When to use

Use when multiple similarity matrices over the same node set (e.g., ONF-based and GIP kernel-based host similarities) must be locally sparsified to K neighbors before SNF-style fusion into a single similarity network.

## When NOT to use

- The similarity matrices come from a single information source and no network fusion is planned — KNN weighting before SNF is specific to integrating multiple host similarities
- The graph is already sparse or kNN-thresholded; re-applying KNN local weighting would discard legitimate edges
- The input is a dissimilarity matrix (e.g., raw Jie Ren d2* values) that has not yet been transformed to similarity

## Inputs

- Host similarity matrix derived from oligonucleotide frequencies (ONF), after transforming Jie Ren dissimilarity values to similarities
- Host similarity matrix derived from Gaussian interaction profile (GIP) kernels
- K-nearest-neighbor count parameter K (per-host neighbor cutoff)

## Outputs

- KNN-weighted status matrix for the ONF host similarity network
- KNN-weighted status matrix for the GIP kernel host similarity network
- Locally weighted matrices ready for SNF iterative diffusion (m=2), feeding the fused matrix written to host_snf_similarity_matrix.csv

## How to apply

For each input similarity matrix (the ONF host similarity obtained by transforming Jie Ren d2* dissimilarities to similarities, and the GIP kernel similarity computed by code/similarity_os.py and code/main.py), build a K-nearest-neighbor status matrix retaining only each host's K most similar neighbors, following the standard SNF status-matrix formulation. Apply the same local weighting to every network so that the iterative SNF diffusion (run with m=2 for the two host similarity types) only exchanges information through dense local neighborhoods rather than full all-to-all matrices. The KNN-weighted matrices then serve as inputs to the SNF iterative diffusion whose diffused status matrices P1 and P2 are averaged element-wise as P = (P1 + P2)/2 to yield the fused host-host similarity matrix. Choose K so each row has exactly K nonzero entries; preserve host labels throughout so the fused output can be written to host_snf_similarity_matrix.csv.

## Related tools

- **NumPy** (Implements the K-nearest-neighbor row-wise selection and construction of the locally weighted status matrices from the ONF and GIP similarity matrices)
- **ILMF-VH** (Repository containing the pipeline (similarity_os.py, main.py for GIP kernel similarity; nrlmf.py for downstream prediction) in which the KNN-weighted networks are fused via SNF) — https://github.com/liudan111/ILMF-VH
- **PyDTI** (Python library (requires Numpy, scikit-learn, Scipy) providing the neighborhood-regularized logistic matrix factorization infrastructure whose conventions the KNN weighting/SNF pipeline follows) — https://github.com/stephenliu0423/PyDTI

## Examples

```
import numpy as np; S_knn = np.where(np.argsort(-S, axis=1) < K, S, 0)  # keep only each host's K most similar neighbors before SNF fusion P = (P1 + P2)/2
```

## Evaluation signals

- Each row of the KNN-weighted matrix contains exactly K nonzero entries, all within the [0, 1] similarity range
- The weighted matrix is square with dimensions equal to the number of hosts and host labels are preserved
- The downstream SNF fusion output (host_snf_similarity_matrix.csv) is square, has values in [0, 1], and matches the structure of the SNF-integrated host-host similarity network reported in the paper
- Both host similarity types (ONF and GIP) are weighted with the same K so the m=2 diffusion processes are comparable
- Fused similarity P equals the element-wise average (P1 + P2)/2 of the two diffused status matrices

## Limitations

- The number of diffusion processes m must match the number of similarity types (m=2 here); adding or removing a similarity source changes the fusion configuration
- ONF inputs are dissimilarities from Jie Ren's tools and must be transformed to similarity before KNN weighting; applying KNN directly to dissimilarities is incorrect
- The host prediction accuracy of ONF-based information alone was reported as less than 40% at the species level, so the KNN weighting should be seen as part of the multi-information fusion rather than a standalone solution

## Evidence

- [methods] KNN local weighting before SNF diffusion: [evidence span withheld — anchor status: not_found]
- [abstract] SNF integration of the two host similarities: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
- [readme] GIP kernel similarity computation scripts: "code/similarity_os.py and code/main.py are used to calculate Gaussian interaction profile kernel similarity between hosts."
- [readme] Dissimilarity-to-similarity transformation requirement: "Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
