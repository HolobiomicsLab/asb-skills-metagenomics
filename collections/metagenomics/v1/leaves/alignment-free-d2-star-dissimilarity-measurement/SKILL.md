---
name: alignment-free-d2-star-dissimilarity-measurement
description: Use when you have raw whole-genome sequences (e.g. NCBI viral and bacterial
  genomes) and need virus-virus or host-host similarity without alignment, to seed
  network-based virus-host prediction methods.
when_to_use_negative:
- Known virus-host associations or alignment-based homology scores already provide
  the required similarity signal (the article notes homology methods depend on whether
  hosts exist in the host genome database)
- The downstream method requires raw interaction profiles rather than ONF similarities
  — e.g. computing Gaussian interaction profile kernel similarity, which ILMF-VH derives
  from known associations via similarity_os.py/main.py instead
- Genomes are highly fragmented or lack sufficient sequence length for stable 6-mer
  frequency estimates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0084
tools:
- name: Jie Ren's d2* oligonucleotide frequency tools
  role: Compute k=6 oligonucleotide frequency vectors and pairwise d2* dissimilarity
    scores between virus and host genomes; results stored under datasets/onf
- name: ILMF-VH
  role: Repository whose datasets/onf stores the ONF measures and whose pipeline consumes
    the transformed similarity matrices for SNF and kernelized logistic matrix factorization
  repo: https://github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Baseline library (NRLMF, CMF, KBMF, NetLapRLS, BLM-NII) executed on the similarity
    inputs for comparison in the ILMF-VH workflow
  repo: https://github.com/stephenliu0423/PyDTI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:2ad76f9b802d87b46b5dbedc887307f91f6bd5411dc952f20f41ea76f3677e79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liudan111/ILMF-VH
  tools:
  - ILMF-VH
  - Jie Ren's d2* oligonucleotide frequency tools
  - PyDTI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# alignment-free-d2-star-dissimilarity-measurement

## Summary

Computes alignment-free d2* oligonucleotide frequency dissimilarities (k=6) between whole-genome sequences of viruses and hosts using Jie Ren's tools, then transforms the dissimilarity matrices into similarity matrices for network construction in the ILMF-VH pipeline.

## When to use

Use when you have raw whole-genome sequences (e.g. NCBI viral and bacterial genomes) and need virus-virus or host-host similarity without alignment, to seed network-based virus-host prediction methods.

## When NOT to use

- Known virus-host associations or alignment-based homology scores already provide the required similarity signal (the article notes homology methods depend on whether hosts exist in the host genome database)
- The downstream method requires raw interaction profiles rather than ONF similarities — e.g. computing Gaussian interaction profile kernel similarity, which ILMF-VH derives from known associations via similarity_os.py/main.py instead
- Genomes are highly fragmented or lack sufficient sequence length for stable 6-mer frequency estimates

## Inputs

- Whole-genome FASTA sequences of viruses (NCBI benchmark viral genomes)
- Whole-genome sequences of hosts (NCBI prokaryotic/bacterial genomes)
- k-mer size parameter k=6

## Outputs

- Pairwise virus-virus d2* dissimilarity matrix transformed into virus similarity matrix Sv (e.g. virus_similarity_matrix.csv)
- Pairwise host-host d2* dissimilarity matrix transformed into host ONF similarity matrix (e.g. host_onf_similarity_matrix.csv)
- datasets/onf folder of ONF measures for viruses and hosts consumed by ILMF-VH

## How to apply

Load the NCBI benchmark viral and bacterial whole-genome sequences and compute 6-mer (k=6) oligonucleotide frequency vectors for each genome using Jie Ren's alignment-free d2* tools. Compute pairwise d2* dissimilarity scores between all virus pairs and all host pairs, following Ahlgren et al. 2017 who showed the d2* measure improves host prediction from metagenomically-derived viral sequences. Because d2* yields dissimilarities, apply a monotone transformation to obtain similarities bounded in [0,1] — the ILMF-VH README explicitly notes 'you should transform to similarity'. In ILMF-VH, the virus-virus similarity matrix defines the virus network directly, while the host ONF similarity is further fused with Gaussian interaction profile kernel similarity via similarity network fusion (SNF) to build the host network.

## Related tools

- **Jie Ren's d2* oligonucleotide frequency tools** (Compute k=6 oligonucleotide frequency vectors and pairwise d2* dissimilarity scores between virus and host genomes; results stored under datasets/onf)
- **ILMF-VH** (Repository whose datasets/onf stores the ONF measures and whose pipeline consumes the transformed similarity matrices for SNF and kernelized logistic matrix factorization) — https://github.com/liudan111/ILMF-VH
- **PyDTI** (Baseline library (NRLMF, CMF, KBMF, NetLapRLS, BLM-NII) executed on the similarity inputs for comparison in the ILMF-VH workflow) — https://github.com/stephenliu0423/PyDTI

## Evaluation signals

- Resulting similarity matrices are symmetric with entries bounded between 0 and 1 after dissimilarity-to-similarity transformation
- Matrix dimensions match the dataset sizes (e.g. 352 viruses / 71 hosts for dataset I; 820 viruses / 2699 hosts for dataset II)
- ONF dissimilarities match those stored in datasets/onf obtained via Jie Ren's tools
- Downstream ILMF-VH accuracy under 5-fold cross-validation is competitive with or better than the five network-based baselines reported in the paper
- Host predictions for crAssphage recover Bacteroides (consistent with prior studies) and suggest Escherichia coli

## Limitations

- No concrete tool names or download locations are specified beyond 'Jie Ren's tools' in the ILMF-VH README
- The dissimilarity must be manually transformed to similarity — the README does not prescribe a specific transformation function
- As an alignment-free sequence measure, ONF similarity depends on hosts of query viruses being represented in the genome database

## Evidence

- [readme] readme tools quote: "Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] readme datasets quote: "datasets/onf including onf measures of viruses and hosts which are obtained by Jie Ren's tools [2]."
- [abstract] abstract virus network quote: "The virus network is constructed based on oligonucleotide frequency measurement"
- [methods] methods workflow quote: [evidence span withheld — anchor status: not_found]
- [readme] readme reference quote: "Alignment-free $d_2^*$ oligonucleotide frequency dissimilarity measure improves prediction of hosts from metagenomically-derived viral sequences."
