---
name: adagrad-optimization
description: Use when training a logistic matrix factorization (e.g. ILMF/NRLMF-style)
  whose latent factors need element-wise adaptive learning rates — e.g. after building
  virus-host heterogeneous networks with c=5 and K1=K2=5 regularization.
when_to_use_negative:
- The goal is only to construct input networks (SNF fusion of host similarities or
  d2* virus dissimilarity); this skill covers the optimizer stage, not network construction.
- The input matrices are raw d2* dissimilarities — the README states 'you should transform
  to similarity' before they can regularize the objective.
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_2269
tools:
- name: nrlmf.py / PyDTI.py (PyDTI)
  role: Reference implementation of the neighborhood-regularized logistic matrix factorization
    objective that is optimized with AdaGrad element-wise learning rates
  repo: https://github.com/stephenliu0423/PyDTI
- name: ILMF-VH code
  role: Virus-host adaptation of the PyDTI ILMF model; its training loop runs AdaGrad
    until convergence and emits ranked virus-host predictions
  repo: https://github.com/liudan111/ILMF-VH
- name: Jie Ren's d2* tools
  role: Compute the oligonucleotide-frequency virus dissimilarity that, once transformed
    to similarity, regularizes the AdaGrad-optimized objective
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:042de7208baf3a6fcebb11ee658411b713b8e5af2d55585af56f250fc0ca6aef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stephenliu0423/PyDTI
  tools:
  - ILMF-VH code
  - Jie Ren's d2* tools
  - nrlmf.py / PyDTI.py (PyDTI)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# adagrad-optimization

## Summary

Element-wise adaptive learning-rate (AdaGrad) optimization of the ILMF-VH kernelized logistic matrix factorization objective (Eq. 11, importance weight c=5) over the virus-host heterogeneous network, iterated until convergence to reconstruct virus-host association scores.

## When to use

Use when training a logistic matrix factorization (e.g. ILMF/NRLMF-style) whose latent factors need element-wise adaptive learning rates — e.g. after building virus-host heterogeneous networks with c=5 and K1=K2=5 regularization.

## When NOT to use

- The goal is only to construct input networks (SNF fusion of host similarities or d2* virus dissimilarity); this skill covers the optimizer stage, not network construction.
- The input matrices are raw d2* dissimilarities — the README states 'you should transform to similarity' before they can regularize the objective.

## Inputs

- Virus-virus similarity matrix (d2* oligonucleotide-frequency dissimilarity computed with Jie Ren's tools, transformed to similarity)
- SNF-fused host-host similarity network (oligonucleotide frequency similarity + Gaussian interaction profile kernel similarity)
- Known virus-host association (bipartite) matrix over 352 viruses x 71 hosts (dataset I) or 820 viruses x 2699 hosts (dataset II)

## Outputs

- Reconstructed virus-host association score matrix
- Ranked virus-host predictions written to virus_host_predictions.csv
- Learned k-dimensional latent factor matrices for viruses and hosts

## How to apply

Initialize the shared k-dimensional latent factor matrices for viruses and hosts with random values. Form the ILMF logistic objective (Eq. 11) with importance weight c=5 so observed virus-host associations outweigh unobserved pairs, then add neighborhood graph regularization over the virus-virus similarity matrix (d2* dissimilarity from Jie Ren's tools, transformed to similarity) and the SNF-fused host-host similarity network with neighborhood sizes K1=K2=5. Before optimizing, correct the negative-sample neighbors per Eqs. 12-13 so neighbors of a known host do not dominate the negative term. Optimize the full objective with AdaGrad element-wise adaptive learning rates until convergence, rather than a single global step size, because gradient magnitudes differ strongly between observed and unobserved pairs. Write the reconstructed association scores and the ranked virus-host predictions to virus_host_predictions.csv. In the reference implementation the optimizer lives inside code/nrlmf.py and is driven by code/PyDTI.py.

## Related tools

- **nrlmf.py / PyDTI.py (PyDTI)** (Reference implementation of the neighborhood-regularized logistic matrix factorization objective that is optimized with AdaGrad element-wise learning rates) — https://github.com/stephenliu0423/PyDTI
- **ILMF-VH code** (Virus-host adaptation of the PyDTI ILMF model; its training loop runs AdaGrad until convergence and emits ranked virus-host predictions) — https://github.com/liudan111/ILMF-VH
- **Jie Ren's d2* tools** (Compute the oligonucleotide-frequency virus dissimilarity that, once transformed to similarity, regularizes the AdaGrad-optimized objective)

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="nr" --cvs=1 --specify-arg=1 --method-opt="c=5 K1=5 K2=5 r=100 lambda_d=0.125 lambda_t=0.125 alpha=0.25 beta=0.125 theta=0.5"
```

## Evaluation signals

- Under five-fold cross validation on the NCBI benchmark, predicted top-ranked hosts match known associations in held-out folds, reproducing ILMF-VH beating the five compared network-based methods.
- Predicted top hosts for crAssphage are consistent with the presumed host Bacteroides, with Escherichia coli surfacing as a further potential host.
- Training objective decreases monotonically and plateaus only at convergence, with element-wise AdaGrad step sizes shrinking adaptively rather than using one global rate.
- Output virus_host_predictions.csv contains ranked virus-host scores with one score per virus-host pair of the input heterogeneous network.
- Sensitivity check: flipping observed pairs to c=5 weight and unobserved to unweighted terms changes the learned factors as expected (observed associations dominate).

## Limitations

- The README requires Python 2.7.9 ('the code is implemented by Python 2.7.9'), so the optimizer code may need porting to Python 3.
- Convergence is monitored but the workflow states no fixed iteration cap or learning-rate value; the user must define a stopping criterion.
- Prediction quality at species level is historically hard — previous ONF-based methods reached less than 40% accuracy at species level, so results should be validated per rank level (genus vs species).

## Evidence

- [methods] Optimize the full objective with AdaGrad element-wise adaptive learning rates until convergence.: [evidence span withheld — anchor status: not_found]
- [methods] Implement the ILMF logistic objective (Eq. 11) using importance weight c=5 to weight observed virus-host associations against unobserved pairs.: [evidence span withheld — anchor status: not_found]
- [intro] we propose a logistic matrix factorization algorithm based on integrating multi-information on the heterogeneous network to predict potential virus-host associations (ILMF-VH).: "we propose a logistic matrix factorization algorithm based on integrating multi-information on the heterogeneous network to predict potential virus-host associations (ILMF-VH)."
- [readme] code/nrlmf.py  are used to predict scores between viruses and hosts and code/PyDTI.py is main function which is applied to execute model simply: "code/nrlmf.py  are used to predict scores between viruses and hosts and code/PyDTI.py is main function which is applied to execute model simply"
- [readme] Note:the code is implemented by Python 2.7.9: "Note:the code is implemented by Python 2.7.9"
