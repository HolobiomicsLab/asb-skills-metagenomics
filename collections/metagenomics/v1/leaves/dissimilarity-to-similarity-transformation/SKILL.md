---
name: dissimilarity-to-similarity-transformation
description: Use when d2* oligonucleotide-frequency dissimilarity values have been
  computed with Jie Ren's tools but the downstream model (ILMF-VH on the virus-host
  heterogeneous network, or SNF host network fusion) requires similarity matrices
  instead.
when_to_use_negative:
- The input matrix is already a similarity (e.g., the Gaussian interaction profile
  kernel similarity computed by similarity_os.py/main.py) — re-transforming it would
  distort the host network used by SNF
- The downstream method consumes dissimilarity directly (e.g., raw d2* distances for
  ranking hosts by oligonucleotide frequency alone) — transformation is only required
  because ILMF-VH's matrix factorization expects similarity
- A different k-mer size or a non-d2* dissimilarity measure is required by the analysis
  protocol; the paper's pipeline is calibrated on d2* with k=6
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: Jie Ren's d2* oligonucleotide frequency tools
  role: Compute the pairwise d2* dissimilarity matrices between virus and host genomes
    (k=6) that are then transformed into similarity matrices
- name: ILMF-VH
  role: Consumer of the transformed virus-virus similarity matrix Sv; its kernelized
    logistic matrix factorization over the virus-host heterogeneous network requires
    similarity input
  repo: github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Framework on which ILMF-VH's logistic matrix factorization code (nrlmf.py
    / PyDTI.py) is built and through which the similarity matrices are consumed
  repo: stephenliu0423/PyDTI
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:3126f99f6d9a9901fe39ea72afbdbef9a22e809247091e022a961fb3e0ec0d6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ILMF-VH
  - ILMF-VH (github.com/liudan111/ILMF-VH)
  - Jie Ren's d2* oligonucleotide frequency tools
  - Jie Ren's oligonucleotide frequency dissimilarity tools
  - Jie Ren's tools
  - Jie Ren's tools (d2* oligonucleotide frequency dissimilarity)
  - PyDTI
  - PyDTI (stephenliu0423/PyDTI)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dissimilarity-to-similarity-transformation

## Summary

Transforms pairwise d2* oligonucleotide-frequency dissimilarity matrices (k=6, computed with Jie Ren's alignment-free tools) into the symmetric, bounded [0,1] similarity matrices that ILMF-VH's kernelized logistic matrix factorization and similarity network fusion require as virus-virus (Sv) and host-host network inputs.

## When to use

Use when d2* oligonucleotide-frequency dissimilarity values have been computed with Jie Ren's tools but the downstream model (ILMF-VH on the virus-host heterogeneous network, or SNF host network fusion) requires similarity matrices instead.

## When NOT to use

- The input matrix is already a similarity (e.g., the Gaussian interaction profile kernel similarity computed by similarity_os.py/main.py) — re-transforming it would distort the host network used by SNF
- The downstream method consumes dissimilarity directly (e.g., raw d2* distances for ranking hosts by oligonucleotide frequency alone) — transformation is only required because ILMF-VH's matrix factorization expects similarity
- A different k-mer size or a non-d2* dissimilarity measure is required by the analysis protocol; the paper's pipeline is calibrated on d2* with k=6

## Inputs

- Whole-genome FASTA sequences of the NCBI benchmark viral and bacterial genomes (820 viruses / 2699 hosts dataset II, or 352 viruses / 71 hosts dataset I)
- Pairwise d2* oligonucleotide-frequency dissimilarity matrices computed with Jie Ren's tools (k-mer size k=6) for virus-virus and host-host pairs
- crAssphage genome JQ995537.1 (for the case-study augmentation as an 821st virus node)

## Outputs

- virus_similarity_matrix.csv — symmetric virus-virus similarity matrix Sv with entries in [0,1]
- host_onf_similarity_matrix.csv — symmetric host-host ONF similarity matrix with entries in [0,1]
- Similarity matrix ready to be appended/updated for crAssphage as an 821st node in the virus similarity network

## How to apply

Compute pairwise d2* oligonucleotide-frequency dissimilarities between whole-genome sequences of the 820/352 benchmark viruses and the bacterial hosts using Jie Ren's alignment-free tools with k-mer size k=6, then apply a monotone transformation that maps each dissimilarity score to a similarity value so that close genomes (low d2*) become high similarity and the original ranking is preserved. The ILMF-VH README explicitly notes that Jie Ren's tools output dissimilarity and that 'you should transform to similarity' before the matrices can feed the kernelized logistic matrix factorization pipeline. Validate each transformed matrix for symmetry and that entries are bounded between 0 and 1, then save as virus_similarity_matrix.csv and host_onf_similarity_matrix.csv. In the host branch, the transformed ONF similarity is subsequently integrated with Gaussian interaction profile kernel similarity via similarity network fusion (SNF neighbor K=5), so the transformation must produce non-negative similarities compatible with that fusion step.

## Related tools

- **Jie Ren's d2* oligonucleotide frequency tools** (Compute the pairwise d2* dissimilarity matrices between virus and host genomes (k=6) that are then transformed into similarity matrices)
- **ILMF-VH** (Consumer of the transformed virus-virus similarity matrix Sv; its kernelized logistic matrix factorization over the virus-host heterogeneous network requires similarity input) — github.com/liudan111/ILMF-VH
- **PyDTI** (Framework on which ILMF-VH's logistic matrix factorization code (nrlmf.py / PyDTI.py) is built and through which the similarity matrices are consumed) — stephenliu0423/PyDTI

## Evaluation signals

- Transformed matrix is symmetric with all similarity values bounded between 0 and 1
- Rank ordering is preserved: pairs with smaller d2* dissimilarity must receive larger similarity values (monotone transformation)
- Downstream ILMF-VH cross-validation reproduces the reported AUPR 0.6305 at k=100 with parameters alpha=0.0625, beta=0.25, lambda=2, gamma=0.25, SNF K=5
- crAssphage case study: augmented 821-node similarity network ranks Escherichia coli first with Bacteroidetes hosts at ranks 4, 44, and 50
- Benchmark accuracy check: top-ranked species-level host prediction accuracy of 58.90% (63.66% top-5 consensus) on the 820-virus benchmark

## Limitations

- The README does not specify the concrete transformation function, only that a transformation is required — the agent must choose and justify an appropriate monotone mapping from d2* dissimilarity to [0,1] similarity
- Dissimilarity-based host inference depends on whether hosts of the query virus exist in the host genome database; sequence-homology-style coverage is limited for novel viruses
- The ILMF-VH codebase is implemented in Python 2.7.9, which may require environment workarounds on modern systems

## Evidence

- [readme] Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity.: "Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [abstract] The virus network is constructed based on oligonucleotide frequency measurement: "The virus network is constructed based on oligonucleotide frequency measurement"
- [abstract] host network constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
- [other] validate that entries are symmetric with similarity values bounded between 0 and 1: [evidence span withheld — anchor status: not_found]
