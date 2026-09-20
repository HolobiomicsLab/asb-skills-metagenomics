---
name: oligonucleotide-frequency-dissimilarity-computation
description: Use when whole-genome FASTA sequences of viruses or host organisms are
  available and you need an alignment-free, k-mer-frequency-based pairwise dissimilarity/similarity
  matrix to seed a network-based prediction method.
when_to_use_negative:
- When host genomes of the query virus are absent from the genome database — sequence-composition
  signals cannot recover associations that lack reference genomes (unlike homology
  limits, composition-based dissimilarity needs genomic context in the network)
- When the goal is genus-level rather than species-level host prediction only — ONF-based
  methods achieved less than 40% species-level accuracy on their own, so fused methods
  like ILMF-VH are preferred
- When a similarity matrix is already available (e.g. datasets/onf pre-computed values)
  — do not recompute dissimilarity from scratch
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Jie Ren's d2* oligonucleotide frequency tools
  role: Compute k-mer (k=6) frequency vectors and pairwise d2* dissimilarity scores
    between virus genomes and between host genomes
- name: ILMF-VH
  role: Distributes pre-computed ONF measures under datasets/onf and consumes the
    transformed virus/host similarity matrices in the heterogeneous network
  repo: github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Framework on which ILMF-VH's kernelized logistic matrix factorization runs
    using the ONF-derived similarity inputs
  repo: stephenliu0423/PyDTI
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_007
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:faf6f4fd310e5db241501c5943fb4576fc1afcb04f75e6fb8159d6ac474b1a3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ILMF-VH
  - Jie Ren's d2* oligonucleotide frequency tools
  - Jie Ren's tools
  - Jie Ren's tools (d2* oligonucleotide frequency dissimilarity)
  - PyDTI
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# oligonucleotide-frequency-dissimilarity-computation

## Summary

Computes alignment-free d2* oligonucleotide frequency dissimilarity between genomes (k=6) to quantify pairwise sequence composition distances, which are then transformed into similarity matrices for the virus-virus and host-host networks in ILMF-VH.

## When to use

Use when whole-genome FASTA sequences of viruses or host organisms are available and you need an alignment-free, k-mer-frequency-based pairwise dissimilarity/similarity matrix to seed a network-based prediction method.

## When NOT to use

- When host genomes of the query virus are absent from the genome database — sequence-composition signals cannot recover associations that lack reference genomes (unlike homology limits, composition-based dissimilarity needs genomic context in the network)
- When the goal is genus-level rather than species-level host prediction only — ONF-based methods achieved less than 40% species-level accuracy on their own, so fused methods like ILMF-VH are preferred
- When a similarity matrix is already available (e.g. datasets/onf pre-computed values) — do not recompute dissimilarity from scratch

## Inputs

- Whole-genome sequences of benchmark viral genomes (FASTA, e.g. 820-virus / 352-virus Edwards et al. datasets from NCBI/RefSeq)
- Whole-genome sequences of bacterial host genomes (FASTA, e.g. 2699 hosts)
- k-mer size parameter k=6

## Outputs

- Pairwise d2* dissimilarity matrices for viruses and hosts
- Transformed symmetric similarity matrices with entries in [0,1]: virus_similarity_matrix.csv (Sv) and host_onf_similarity_matrix.csv
- ONF similarity profiles stored under datasets/onf in the ILMF-VH repository

## How to apply

Load the whole-genome sequences of the benchmark viral and bacterial genomes from the NCBI dataset files. Compute oligonucleotide frequency vectors for each genome using k-mer size k=6 with Jie Ren's alignment-free tools (as shipped in the datasets/onf folder of the ILMF-VH repository), then compute pairwise d2* dissimilarity scores between all virus genome pairs and all host genome pairs. Because Jie Ren's tools output dissimilarity while the downstream pipeline requires similarity, apply a monotone transformation mapping each d2* dissimilarity to a similarity value in [0,1]. Write the transformed matrices as virus_similarity_matrix.csv and host_onf_similarity_matrix.csv; these feed SNF fusion (with the GIP kernel) and the virus-virus similarity network used by kernelized logistic matrix factorization.

## Related tools

- **Jie Ren's d2* oligonucleotide frequency tools** (Compute k-mer (k=6) frequency vectors and pairwise d2* dissimilarity scores between virus genomes and between host genomes)
- **ILMF-VH** (Distributes pre-computed ONF measures under datasets/onf and consumes the transformed virus/host similarity matrices in the heterogeneous network) — github.com/liudan111/ILMF-VH
- **PyDTI** (Framework on which ILMF-VH's kernelized logistic matrix factorization runs using the ONF-derived similarity inputs) — stephenliu0423/PyDTI

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="nr" --specify-arg=1 --method-opt="r=100"  # after loading datasets/onf d2* measures transformed to similarity
```

## Evaluation signals

- Both output matrices must be square with dimensions equal to the number of viruses (e.g. 820) or hosts (e.g. 2699)
- Similarity matrices must be symmetric with entries bounded between 0 and 1 after transformation
- Downstream benchmark must reproduce reported ILMF-VH accuracies (58.90% direct top-ranked, 63.66% top-5 consensus) on the 820-virus/2699-host benchmark under five-fold cross validation
- crAssphage case study must rank Escherichia coli first among 2699 hosts with Bacteroidetes hosts at ranks 4, 44, and 50
- The transformation step must be verified — Jie Ren's tools output dissimilarity and the pipeline explicitly requires transforming to similarity before use

## Limitations

- ONF similarity alone yields less than 40% species-level host prediction accuracy, motivating fusion with GIP kernels via SNF
- Jie Ren's tools produce dissimilarity, not similarity; skipping the monotone transformation step breaks the downstream pipeline
- Sequence composition signals depend on whether hosts of the query virus exist in the host genome database

## Evidence

- [methods] d2* dissimilarity computation with k=6: [evidence span withheld — anchor status: not_found]
- [readme] Jie Ren's tools and transformation requirement: "Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] Pre-computed ONF measures shipped with the code: "datasets/onf including onf measures of viruses and hosts which are obtained by Jie Ren's tools"
- [abstract] Virus network built from oligonucleotide frequency: "The virus network is constructed based on oligonucleotide frequency measurement"
