---
name: virus-similarity-network-augmentation
description: Use when a newly sequenced phage genome (e.g., an NCBI accession not
  in the 820-virus RefSeq benchmark) must be added as a node to an existing virus
  similarity network before virus-host association prediction.
when_to_use_negative:
- The query virus's host is absent from the host genome database — the article notes
  homology/similarity-based host inference depends on hosts existing in the database
- The task is de novo host prediction from metagenomic contigs without a genome that
  can be aligned to the 820-virus oligonucleotide framework
- Abundance-profile co-variation (not sequence-based network augmentation) is the
  intended signal, since co-variation may reflect time-delayed dynamics rather than
  real interaction
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: Jie Ren's tools (d2*)
  role: Compute d2* oligonucleotide frequency dissimilarity between the query virus
    and the 820 benchmark viruses; output must be transformed to similarity
- name: ILMF-VH
  role: Kernelized logistic matrix factorization that consumes the augmented virus
    similarity network within the virus-host heterogeneous network to score query-virus
    to host associations
  repo: https://github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Underlying Python 2.7 library providing matrix factorization and baseline
    DTI methods (NRLMF, CMF, KBMF, NetLapRLS, BLM-NII) on which ILMF-VH prediction
    code is executed
  repo: https://github.com/stephenliu0423/PyDTI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:6e3608bb46200f9de52b59677e94614c5aedf90c4a33f2b797fa613f4ab88c84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liudan111/ILMF-VH
  tools:
  - ILMF-VH
  - Jie Ren's tools (d2*)
  - PyDTI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# virus-similarity-network-augmentation

## Summary

Appends a new query virus (e.g., crAssphage, JQ995537.1) as an extra node in the 820-virus oligonucleotide-frequency similarity network by computing d2* dissimilarity via Jie Ren's tools and transforming it to similarity, so kernelized logistic matrix factorization (ILMF-VH) can score it against all 2699 candidate hosts.

## When to use

Use when a newly sequenced phage genome (e.g., an NCBI accession not in the 820-virus RefSeq benchmark) must be added as a node to an existing virus similarity network before virus-host association prediction.

## When NOT to use

- The query virus's host is absent from the host genome database — the article notes homology/similarity-based host inference depends on hosts existing in the database
- The task is de novo host prediction from metagenomic contigs without a genome that can be aligned to the 820-virus oligonucleotide framework
- Abundance-profile co-variation (not sequence-based network augmentation) is the intended signal, since co-variation may reflect time-delayed dynamics rather than real interaction

## Inputs

- Query phage genome in FASTA format (e.g., NCBI accession JQ995537.1, crAssphage)
- Existing 820-virus similarity network derived from d2* oligonucleotide frequency dissimilarity (datasets/onf from Jie Ren's tools)
- Virus-host heterogeneous network components: host-host similarity network (SNF-fused from oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity) and known virus-host associations

## Outputs

- Augmented virus similarity network with the query virus as node 821
- Ranked list of predicted hosts for the query virus (e.g., crAssphage_host_predictions.csv with 2699 hosts ordered by association score)

## How to apply

Download the query virus genome (e.g., crAssphage accession JQ995537.1) and compute d2* oligonucleotide frequency dissimilarity against the 820 benchmark viruses using Jie Ren's tools, which produce 'onf' measures stored in datasets/onf. Because the tools output dissimilarity, transform the values to similarity scores before network integration. Append the query as an 821st row/column in the virus similarity network, then supply the augmented network to the ILMF-VH kernelized logistic matrix factorization (built on PyDTI, e.g., via code/nrlmf.py and code/PyDTI.py) together with the host-host similarity network built by similarity network fusion. Rank all 2699 candidate hosts by predicted association score; for crAssphage, a correct run places Escherichia coli at rank 1 with Bacteroidetes hosts at ranks 4, 44, and 50.

## Related tools

- **Jie Ren's tools (d2*)** (Compute d2* oligonucleotide frequency dissimilarity between the query virus and the 820 benchmark viruses; output must be transformed to similarity)
- **ILMF-VH** (Kernelized logistic matrix factorization that consumes the augmented virus similarity network within the virus-host heterogeneous network to score query-virus to host associations) — https://github.com/liudan111/ILMF-VH
- **PyDTI** (Underlying Python 2.7 library providing matrix factorization and baseline DTI methods (NRLMF, CMF, KBMF, NetLapRLS, BLM-NII) on which ILMF-VH prediction code is executed) — https://github.com/stephenliu0423/PyDTI

## Examples

```
python code/nrlmf.py --dataset='dataset II' --predict-num=50  # after augmenting the virus similarity network with crAssphage d2* similarities from Jie Ren's tools
```

## Evaluation signals

- The transformed similarity matrix for the query virus has 820 entries matching the benchmark virus count, and the augmented network has dimension 821 (820 viruses + crAssphage)
- Dissimilarity-to-similarity transformation was applied: Jie Ren's tools output dissimilarity, and the network requires similarity values
- Ranked output contains all 2699 candidate hosts from dataset II
- For the crAssphage case study, Escherichia coli holds rank 1 and Bacteroidetes hosts appear at ranks 4, 44, and 50 within the top 50
- Predicted host for crAssphage is consistent with the presumed Bacteroides host from previous studies, with Escherichia coli as an additional potential host

## Limitations

- Similarity/homology-based host prediction depends on whether hosts of the query virus exist in the host genome database
- Case-study ranks may vary if the benchmark (dataset II: 2699 hosts, 820 viruses from RefSeq 7/25/2015) is updated or re-downloaded
- Code is implemented in Python 2.7.9; KBMF requires pymatbridge and Matlab, adding environment constraints

## Evidence

- [readme] Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity.: "Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] datasets/onf including onf measures of viruses and hosts which are obtained by Jie Ren's tools [2].: "datasets/onf including onf measures of viruses and hosts which are obtained by Jie Ren's tools [2]."
- [abstract] The virus network is constructed based on oligonucleotide frequency measurement: "The virus network is constructed based on oligonucleotide frequency measurement"
- [intro] the host of crAssphage inferred by our algorithm includes putative host Bacteroides obtained from previous studies [9, 19], and another potential host Escherichia coli is also suggested.: "the host of crAssphage inferred by our algorithm includes putative host Bacteroides obtained from previous studies [9, 19], and another potential host Escherichia coli is also suggested."
- [readme] datasets/dataset II including 2699 hosts and 820 viruses which is downloaded from RefSeq on 7/25/2015[1].: "datasets/dataset II including 2699 hosts and 820 viruses which is downloaded from RefSeq on 7/25/2015[1]."
