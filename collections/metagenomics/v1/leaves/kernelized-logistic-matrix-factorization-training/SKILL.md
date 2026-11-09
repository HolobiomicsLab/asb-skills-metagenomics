---
name: kernelized-logistic-matrix-factorization-training
description: Use when predicting virus-host associations from a known association
  matrix plus virus/host similarity networks, e.g. cross-validation benchmarking on
  352-virus/71-host data or host ranking for a novel virus like crAssphage.
when_to_use_negative:
- The query virus's hosts have no genomes in the host database and no virus similarity
  can be computed, since sequence-homology and network inputs depend on hosts of the
  query virus existing in the database
- You only have abundance profile co-variation data without genomic sequence, because
  co-variation does not necessarily represent real interaction due to time delays
  in virus-host dynamics
- The task is drug-target interaction prediction with the original PyDTI datasets
  (nr, gpcr, ic, e) rather than virus-host association
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: ILMF-VH
  role: Implements kernelized logistic matrix factorization on the heterogeneous virus-host
    network; code/nrlmf.py predicts virus-host scores and code/PyDTI.py executes the
    model
  repo: https://github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Framework providing baseline methods KBMF2K, BLM-NII, NetLapRLS, and CMF used
    for five-fold cross-validation comparison against ILMF-VH
  repo: https://github.com/stephenliu0423/PyDTI
- name: Jie Ren's d2* oligonucleotide frequency tools
  role: Compute d2* oligonucleotide frequency dissimilarity between virus genomes,
    which must be transformed to similarity for the virus network
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:d615e1eb4aee240c60f626bf9267aa5bdc1bf88103f21217faa380552256c18d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liudan111/ILMF-VH
  tools:
  - ILMF-VH
  - Jie Ren's d2* oligonucleotide frequency tools
  - PyDTI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# kernelized-logistic-matrix-factorization-training

## Summary

Trains ILMF-VH, a kernelized logistic matrix factorization model, on a virus-host heterogeneous network built from oligonucleotide-frequency virus similarity, SNF-fused host similarity, and known virus-host associations to predict potential virus-host interactions.

## When to use

Use when predicting virus-host associations from a known association matrix plus virus/host similarity networks, e.g. cross-validation benchmarking on 352-virus/71-host data or host ranking for a novel virus like crAssphage.

## When NOT to use

- The query virus's hosts have no genomes in the host database and no virus similarity can be computed, since sequence-homology and network inputs depend on hosts of the query virus existing in the database
- You only have abundance profile co-variation data without genomic sequence, because co-variation does not necessarily represent real interaction due to time delays in virus-host dynamics
- The task is drug-target interaction prediction with the original PyDTI datasets (nr, gpcr, ic, e) rather than virus-host association

## Inputs

- Known virus-host association matrix (binary, from NCBI benchmark viral genomes annotation)
- Virus-virus similarity network derived from d2* oligonucleotide frequency dissimilarity (Jie Ren's tools)
- Host-host similarity network (oligonucleotide frequency similarity fused with Gaussian interaction profile kernel similarity via SNF)
- Datasets: dataset I (352 viruses, 71 hosts) and dataset II (820 viruses, 2699 hosts from RefSeq 7/25/2015)

## Outputs

- Predicted virus-host association score matrix
- Five-fold cross-validation AUC/AUPR per method (e.g. cv_results_table.csv)
- Ranked list of candidate hosts for a query virus (e.g. crAssphage_host_predictions.csv)

## How to apply

Construct a virus similarity network from d2* oligonucleotide frequency dissimilarities computed with Jie Ren's tools, transforming dissimilarity to similarity, and a host similarity network by fusing oligonucleotide frequency similarity with Gaussian interaction profile kernel similarity via similarity network fusion (SNF). Connect the virus and host networks with the known association matrix to form the heterogeneous network, then run kernelized logistic matrix factorization using code/nrlmf.py with code/PyDTI.py as the main executor from the liudan111/ILMF-VH repository (Python 2.7.9). For benchmarking, split known associations into five folds, aggregate held-out predictions, and compute AUC and AUPR per method; expect ILMF-VH to reach AUC 0.9202 and AUPR 0.6243 on dataset I. For novel-virus prediction, append the query virus as an extra node in the virus similarity network and rank candidate hosts by predicted score.

## Related tools

- **ILMF-VH** (Implements kernelized logistic matrix factorization on the heterogeneous virus-host network; code/nrlmf.py predicts virus-host scores and code/PyDTI.py executes the model) — https://github.com/liudan111/ILMF-VH
- **PyDTI** (Framework providing baseline methods KBMF2K, BLM-NII, NetLapRLS, and CMF used for five-fold cross-validation comparison against ILMF-VH) — https://github.com/stephenliu0423/PyDTI
- **Jie Ren's d2* oligonucleotide frequency tools** (Compute d2* oligonucleotide frequency dissimilarity between virus genomes, which must be transformed to similarity for the virus network)

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="datasetI" --cvs=1 --specify-arg=1 --method-opt="r=100"
```

## Evaluation signals

- Five-fold cross-validation AUC and AUPR for ILMF-VH match Table 2 values (AUC 0.9202, AUPR 0.6243 on dataset I with 352 viruses)
- ILMF-VH outperforms all five baselines (LMFH-VH, KBMF, BLM-NII, NetLapRLS, CMF) on both AUC and AUPR
- crAssphage case study: Escherichia coli (Proteobacteria) ranks 1st among 2699 candidate hosts, with Bacteroidetes hosts at ranks 4, 44, and 50 in the top 50
- Virus similarity matrix is symmetric and contains transformed similarity (not raw d2* dissimilarity) values
- Cross-validation split covers all known associations exactly once across the five folds with no training-set leakage

## Limitations

- Homology-based identification of hosts for new viruses is limited because it depends on whether hosts of the query virus exist in the host genome database
- Abundance profile co-variation does not necessarily represent real interaction because there is usually a time delay in dynamic interactions between viruses and hosts
- Code is implemented in Python 2.7.9, and KBMF2K requires pymatbridge plus external Matlab code from M. Gonen
- Host prediction accuracy at species level was previously less than 40% for ONF-based methods, a difficulty this model aims to address but which constrains expected species-level accuracy

## Evidence

- [abstract] a kernelized logistic matrix factorization with integrating different information to predict potential virus-host associations on the heterogeneous network (ILMF-VH): "a kernelized logistic matrix factorization with integrating different information to predict potential virus-host associations on the heterogeneous network (ILMF-VH)"
- [abstract] the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
- [readme] Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity.: "Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] code/nrlmf.py  are used to predict scores between viruses and hosts and code/PyDTI.py is main function which is applied to execute model simply: "code/nrlmf.py  are used to predict scores between viruses and hosts and code/PyDTI.py is main function which is applied to execute model simply"
- [results] ILMF-VH achieved the best performance among the six methods, with an AUC value of 0.9202 and an AUPR value of 0.6243 on the dataset containing 352 viruses.: [evidence span withheld — anchor status: not_found]
