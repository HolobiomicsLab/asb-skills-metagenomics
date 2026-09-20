---
name: host-similarity-matrix-construction
description: Use when you have two complementary host similarity sources (e.g., genome
  k-mer ONF similarity and GIP kernel similarity from known virus-host associations)
  and need a single square host-host similarity matrix to feed a heterogeneous virus-host
  network model.
when_to_use_negative:
- You only have one host similarity source (e.g., only ONF or only GIP) — SNF fusion
  requires at least two input similarity networks to diffuse between
- Your goal is direct sequence homology-based host assignment for a query virus whose
  candidate hosts are absent from the host genome database — the article notes such
  homology methods are limited because they 'depend on whether hosts of the query
  virus exist in the host genome database'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Jie Ren's d2* oligonucleotide frequency tools
  role: Compute pairwise d2* oligonucleotide frequency dissimilarity between host
    genomes (k=6); outputs must be transformed from dissimilarity to similarity before
    fusion
- name: NumPy
  role: Matrix operations for KNN weighting, SNF diffusion (m=2 iterations), and element-wise
    averaging P = (P1 + P2)/2
- name: ILMF-VH codebase
  role: Provides similarity_os.py and main.py to compute the Gaussian interaction
    profile kernel similarity between hosts, one of the two SNF inputs
  repo: github.com/liudan111/ILMF-VH
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:feec5f77f431b4d8022068398d1246d5fb059e4907c983ad308843fb37543b73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ILMF-VH codebase
  - Jie Ren's d2* oligonucleotide frequency tools
  - NumPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# host-similarity-matrix-construction

## Summary

Constructs a fused host-host similarity network for ILMF-VH by combining oligonucleotide frequency (d2*, k=6) similarity with Gaussian interaction profile kernel similarity using similarity network fusion (SNF), replacing a single-sequence-similarity host network with a multi-source integrated one.

## When to use

Use when you have two complementary host similarity sources (e.g., genome k-mer ONF similarity and GIP kernel similarity from known virus-host associations) and need a single square host-host similarity matrix to feed a heterogeneous virus-host network model.

## When NOT to use

- You only have one host similarity source (e.g., only ONF or only GIP) — SNF fusion requires at least two input similarity networks to diffuse between
- Your goal is direct sequence homology-based host assignment for a query virus whose candidate hosts are absent from the host genome database — the article notes such homology methods are limited because they 'depend on whether hosts of the query virus exist in the host genome database'

## Inputs

- Whole-genome sequences of host (prokaryotic/bacterial) genomes from the NCBI benchmark dataset (FASTA)
- d2* host-host dissimilarity matrix from Jie Ren's ONF tools (k=6), to be transformed to similarity
- Known virus-host association matrix for computing Gaussian interaction profile kernel similarity
- SNF parameters: K-nearest-neighbor count K and number of similarity networks m=2

## Outputs

- Fused host-host similarity matrix (host_snf_similarity_matrix.csv), square, dimensions equal to number of hosts (e.g., 71 for dataset I), values in [0,1], host labels preserved
- Intermediate virus/host similarity CSVs (host_onf_similarity_matrix.csv)

## How to apply

First compute the host-host ONF similarity matrix by calculating pairwise d2* oligonucleotide frequency dissimilarity with Jie Ren's alignment-free tools (k-mer size k=6) and applying a monotone transformation mapping dissimilarity to similarity. Independently compute the host-host Gaussian interaction profile (GIP) kernel similarity (in the ILMF-VH code via code/similarity_os.py and code/main.py). Apply K-nearest-neighbor local weighting (standard SNF status matrices) so each host's K most similar neighbors dominate, then run the SNF iterative diffusion with m=2 networks for m=2 iterations, each network's status matrix updated from the other's similarity structure. The final fused matrix is the element-wise average P = (P1 + P2)/2 of the two diffused status matrices, written with host labels preserved.

## Related tools

- **Jie Ren's d2* oligonucleotide frequency tools** (Compute pairwise d2* oligonucleotide frequency dissimilarity between host genomes (k=6); outputs must be transformed from dissimilarity to similarity before fusion)
- **NumPy** (Matrix operations for KNN weighting, SNF diffusion (m=2 iterations), and element-wise averaging P = (P1 + P2)/2)
- **ILMF-VH codebase** (Provides similarity_os.py and main.py to compute the Gaussian interaction profile kernel similarity between hosts, one of the two SNF inputs) — github.com/liudan111/ILMF-VH

## Examples

```
P_fused = (P_onf_diffused + P_gip_diffused) / 2  # SNF: m=2 networks, element-wise average of diffused status matrices, written to host_snf_similarity_matrix.csv
```

## Evaluation signals

- Output host_snf_similarity_matrix.csv exists and is square with dimensions equal to the number of hosts (71 hosts for dataset I, 2699 for dataset II)
- All fused similarity values lie in the [0, 1] range
- The matrix is symmetric (similarity of host i to host j equals host j to host i)
- The fused matrix structure matches the SNF-integrated host-host similarity network reported in the paper
- ONF inputs derived from Jie Ren's tools are dissimilarities and have been monotonically transformed to similarities before fusion (README explicitly warns of this transformation step)

## Limitations

- Jie Ren's tools yield dissimilarity scores, not similarities — failing to transform them before SNF will corrupt the fusion
- Host prediction depends on whether hosts of the query virus exist in the host genome database; hosts missing from NCBI's prokaryotic genome database are excluded (dataset I only includes viruses whose specific host genome appeared there)
- The SNF setup is specific to m=2 host similarity types; adding or removing a similarity source changes the diffusion configuration

## Evidence

- [abstract] Host network constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
- [intro] SNF integrates multiple host information sources: "applies similar network fusion (SNF) to integrate multiple host information for constructing the host-host similarity network"
- [readme] README warns that Jie Ren's tools output dissimilarity that must be transformed to similarity: "Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] README identifies the scripts that compute the GIP kernel similarity between hosts: "code/similarity_os.py and code/main.py are used to calculate Gaussian interaction profile kernel similarity between hosts."
