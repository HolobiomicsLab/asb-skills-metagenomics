---
name: virus-host-association-prediction
description: Use when you need to rank candidate hosts for uncultured phage/virus
  genomes from sequence data alone, given a known virus-host association matrix plus
  virus-virus and host-host similarity networks.
when_to_use_negative:
- The query virus's host is absent from the host genome database — sequence-homology/oligonucleotide-based
  similarity signals are limited in this case, as noted in the article's limitations
- You already have direct biological evidence of virus-host interaction (e.g. cultured
  infection data) — a prediction method is unnecessary
- The task is genus-level-only prediction with an ONF-based method already sufficient;
  ILMF-VH targets improving species-level accuracy, which prior ONF methods achieved
  at less than 40%
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: ILMF-VH
  role: Main implementation of the kernelized logistic matrix factorization with neighborhood
    regularization (nrlmf.py / PyDTI.py) that predicts virus-host scores on the heterogeneous
    network
  repo: https://github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Framework providing the five baseline network-based methods (LMFH-VH, KBMF2K,
    BLM-NII, NetLapRLS, CMF) and cross-validation harness used for comparison under
    five-fold CV
  repo: https://github.com/stephenliu0423/PyDTI
- name: Jie Ren's tools (d2*)
  role: Compute oligonucleotide-frequency dissimilarity between viruses (and hosts);
    outputs must be transformed to similarity before network construction
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:91283bc5d477ba3ad7faa04b8db5674a292c693d49c13d6053a5e19581f95866
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liudan111/ILMF-VH
  tools:
  - ILMF-VH
  - Jie Ren's tools
  - Jie Ren's tools (d2*)
  - PyDTI
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# virus-host-association-prediction

## Summary

Culture-free prediction of bacteriophage hosts using ILMF-VH: kernelized logistic matrix factorization with neighborhood regularization applied to a virus-host heterogeneous network built from oligonucleotide-frequency (d2*) virus similarity, SNF-fused host similarity, and known associations.

## When to use

Use when you need to rank candidate hosts for uncultured phage/virus genomes from sequence data alone, given a known virus-host association matrix plus virus-virus and host-host similarity networks.

## When NOT to use

- The query virus's host is absent from the host genome database — sequence-homology/oligonucleotide-based similarity signals are limited in this case, as noted in the article's limitations
- You already have direct biological evidence of virus-host interaction (e.g. cultured infection data) — a prediction method is unnecessary
- The task is genus-level-only prediction with an ONF-based method already sufficient; ILMF-VH targets improving species-level accuracy, which prior ONF methods achieved at less than 40%

## Inputs

- Virus genome sequences (e.g. 820 RefSeq viral genomes downloaded 7/25/2015, or NCBI accession lists)
- Host/prokaryotic genome sequences (e.g. 2699 hosts from dataset II, or 71 hosts in dataset I)
- Known virus-host association matrix (extracted via 'isolate host =' or 'host' fields from NCBI annotation files)
- d2* oligonucleotide-frequency dissimilarity measures between viruses/hosts (from the datasets/onf folder computed with Jie Ren's tools)
- Gaussian interaction profile kernel similarities between hosts for SNF fusion

## Outputs

- Ranked virus-host association scores per virus (e.g. virus_host_predictions.csv, crAssphage_host_predictions.csv)
- Per-method AUC/AUPR cross-validation results table (cv_results_table.csv)
- Species-level host prediction accuracies for direct top-ranked and top-5 consensus predictions (benchmark_results.csv)

## How to apply

Build a virus-virus similarity network from oligonucleotide-frequency dissimilarities computed with Jie Ren's d2* tools, transforming dissimilarity values to similarity scores as the pipeline requires. Build the host-host similarity network by integrating oligonucleotide-frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion (SNF). Connect the two networks through known virus-host associations to form the heterogeneous network, then run kernelized logistic matrix factorization (ILMF-VH, built on the PyDTI framework) with logistic objective weight c=5, latent dimension r=100, neighborhood sizes K1=K2=5, applying the negative-sample neighbor correction of Eqs. 12-13 so potential unknown associations still influence training, optimized with AdaGrad until convergence. Rank all candidate hosts per virus by predicted association score and evaluate under five-fold cross validation; the reference performance is 58.90% species-level top-1 accuracy (63.66% top-5 consensus) on the 820-virus/2699-host benchmark and AUC 0.9202 / AUPR 0.6243 on the 352-virus/71-host dataset.

## Related tools

- **ILMF-VH** (Main implementation of the kernelized logistic matrix factorization with neighborhood regularization (nrlmf.py / PyDTI.py) that predicts virus-host scores on the heterogeneous network) — https://github.com/liudan111/ILMF-VH
- **PyDTI** (Framework providing the five baseline network-based methods (LMFH-VH, KBMF2K, BLM-NII, NetLapRLS, CMF) and cross-validation harness used for comparison under five-fold CV) — https://github.com/stephenliu0423/PyDTI
- **Jie Ren's tools (d2*)** (Compute oligonucleotide-frequency dissimilarity between viruses (and hosts); outputs must be transformed to similarity before network construction)

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="dataset II" --cvs=1 --specify-arg=1 --method-opt="c=5 K1=5 K2=5 r=100"
```

## Evaluation signals

- Reproduced five-fold CV AUC/AUPR for ILMF-VH matches Table 2: AUC 0.9202 and AUPR 0.6243 on the 352-virus dataset, better than all five baselines
- Species-level accuracy on the 820-virus/2699-host benchmark reaches 58.90% for top-ranked prediction and 63.66% for the top-5 consensus method
- crAssphage case study: Escherichia coli holds rank 1 among 2699 candidate hosts and Bacteroidetes hosts appear at ranks 4, 44, and 50 of the top 50
- Predicted top-ranked hosts match known associations in held-out cross-validation folds
- Dissimilarity values from Jie Ren's tools have been correctly transformed to similarity before feeding the network (README note: 'you should transform to similarity')

## Limitations

- Code is implemented in Python 2.7.9; KBMF2K additionally requires pymatbridge (a Python-Matlab bridge) and external Matlab code from M. Gonen
- Sequence homology-style signals depend on whether hosts of the query virus exist in the host genome database
- Abundance profile co-variation does not necessarily represent real interaction because of time delays in dynamic virus-host interactions
- Jie Ren's tools yield dissimilarity, not similarity; failing to transform values breaks the network inputs
- The README does not name the concrete tool binaries for dissimilarity computation beyond 'Jie Ren's tools'

## Evidence

- [abstract] virus network constructed from oligonucleotide frequency; heterogeneous network + kLMF: "The virus network is constructed based on oligonucleotide frequency measurement"
- [abstract] SNF fuses host information sources: "the host network is constructed by integrating oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity through similarity network fusion"
- [methods] d2* dissimilarity must be transformed to similarity: "Jie Ren's tools is used to get dissimilarity between viruses"
- [readme] Dataset composition (dataset I and II): "datasets/dataset II including 2699 hosts and 820 viruses which is downloaded from RefSeq on 7/25/2015"
- [readme] onf measures provenance and transformation requirement: "datasets/onf including onf measures of viruses and hosts which are obtained by Jie Ren's tools [2]"
