---
name: hyperparameter-sensitivity-analysis
description: Use when a model such as ILMF-VH has multiple tunable parameters (k,
  alpha, beta, lambda, gamma, K) and you must sweep each while fixing the others,
  scoring every setting by mean AUPR under five-fold cross-validation.
when_to_use_negative:
- The goal is comparing ILMF-VH against the five baseline network-based methods (KBMF,
  NetLapRLS, BLM-NII, CMF) with fixed settings rather than tuning parameters
- No ground-truth virus-host association matrix is available, so five-fold cross-validated
  AUPR cannot be computed
- The input is a parameter-tuning request for a model without neighborhood regularization
  or SNF components, where alpha, beta, and K are undefined
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ILMF-VH
  role: Kernelized logistic matrix factorization model whose parameters (k, alpha,
    beta, lambda, gamma, K) are swept and scored by five-fold cross-validated AUPR
  repo: github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Baseline library providing the logistic matrix factorization framework (nrlmf.py,
    PyDTI.py) on which ILMF-VH is built; PyDTI.py supports choosing optimal parameters
    via --specify-arg=0
  repo: stephenliu0423/PyDTI
- name: Jie Ren's oligonucleotide frequency dissimilarity tools
  role: Compute the d2* dissimilarity between viruses that must be transformed to
    similarity before the sensitivity sweeps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:62ca01551ee6178ebfb8b23f762754060c5014fdfd9195a2561cecbbfbe45414
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ILMF-VH
  - ILMF-VH (github.com/liudan111/ILMF-VH)
  - Jie Ren's oligonucleotide frequency dissimilarity tools
  - PyDTI
  - PyDTI (stephenliu0423/PyDTI)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# hyperparameter-sensitivity-analysis

## Summary

Sweeps the tunable parameters of a predictive model (here, ILMF-VH's matrix factorization factor k, neighborhood regularization alpha/beta, Gaussian kernel lambda, learning rate gamma, and SNF neighbor K) and records mean AUPR under five-fold cross-validation to identify optima and produce sensitivity trend panels.

## When to use

Use when a model such as ILMF-VH has multiple tunable parameters (k, alpha, beta, lambda, gamma, K) and you must sweep each while fixing the others, scoring every setting by mean AUPR under five-fold cross-validation.

## When NOT to use

- The goal is comparing ILMF-VH against the five baseline network-based methods (KBMF, NetLapRLS, BLM-NII, CMF) with fixed settings rather than tuning parameters
- No ground-truth virus-host association matrix is available, so five-fold cross-validated AUPR cannot be computed
- The input is a parameter-tuning request for a model without neighborhood regularization or SNF components, where alpha, beta, and K are undefined

## Inputs

- NCBI benchmark virus-host association matrix (datasets/dataset I with 71 hosts and 352 viruses; dataset II with 2699 hosts and 820 viruses)
- Virus-virus similarity matrix obtained by transforming Jie Ren's d2* oligonucleotide-frequency dissimilarity (datasets/onf) to similarity
- Host-host similarity networks (oligonucleotide frequency similarity plus Gaussian interaction profile kernel similarity, fused via SNF)
- ILMF-VH codebase (built on PyDTI-style logistic matrix factorization, Python 2.7.9)

## Outputs

- Mean AUPR (five-fold cross-validation) per value for each swept parameter (k, alpha, beta, lambda, gamma, K)
- Five AUPR-versus-parameter trend panels reproducing Figures S1-S5 of Additional file 1
- Optimal parameter settings: k=100, alpha=0.0625, beta=0.25, lambda=2, gamma=0.25, K=5, with peak mean AUPR 0.6305 at k=100

## How to apply

Load the NCBI benchmark virus-host association data and the virus similarity matrix derived by converting Jie Ren's d2* oligonucleotide-frequency dissimilarity to similarity. Using the ILMF-VH codebase (a PyDTI-style logistic matrix factorization), sweep the latent factor number k while fixing alpha, beta, lambda, gamma, and K, recording mean AUPR under five-fold cross-validation at each value; then repeat the one-parameter sweep for alpha and beta (neighborhood regularization), lambda (Gaussian interaction profile kernel inverse variance), gamma (learning rate), and the similarity network fusion neighbor number K. Plot AUPR versus each parameter value to reproduce the five trend panels corresponding to Figures S1-S5. Accept the sweep only if the argmax values reproduce the reported settings: k=100 (AUPR 0.6305), alpha=0.0625, beta=0.25, lambda=2, gamma=0.25, and K=5. The rationale is that ILMF-VH's performance depends on each parameter independently, so isolating sweeps reveal the sensitivity profile and justify the settings used for method comparison.

## Related tools

- **ILMF-VH** (Kernelized logistic matrix factorization model whose parameters (k, alpha, beta, lambda, gamma, K) are swept and scored by five-fold cross-validated AUPR) — github.com/liudan111/ILMF-VH
- **PyDTI** (Baseline library providing the logistic matrix factorization framework (nrlmf.py, PyDTI.py) on which ILMF-VH is built; PyDTI.py supports choosing optimal parameters via --specify-arg=0) — stephenliu0423/PyDTI
- **Jie Ren's oligonucleotide frequency dissimilarity tools** (Compute the d2* dissimilarity between viruses that must be transformed to similarity before the sensitivity sweeps)

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="nr" --cvs=1 --specify-arg=1 --method-opt="r=100"
```

## Evaluation signals

- Mean AUPR under five-fold cross-validation reaches its maximum at k=100 with AUPR 0.6305
- Swept optima match the reported settings: alpha=0.0625, beta=0.25, lambda=2, gamma=0.25, and K=5
- Figure file exists and contains exactly five AUPR-versus-parameter trend panels, each with the optimum value marked, mirroring Figures S1-S5 of Additional file 1
- Each sweep fixes all other parameters while varying one, per the one-factor-at-a-time design
- Every AUPR point is the average over the five cross-validation folds, not a single fold

## Limitations

- Jie Ren's tools output dissimilarity, not similarity; the user must perform the transformation to similarity before use
- Code is implemented in Python 2.7.9, and KBMF requires a Matlab bridge (pymatbridge) when used as a baseline
- Sequence homology and co-variation signals depend on whether hosts of the query virus exist in the host genome database, which constrains the association data available for validation

## Evidence

- [readme] Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity.: "Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [intro] applies similar network fusion (SNF) to integrate multiple host information for constructing the host-host similarity network: "applies similar network fusion (SNF) to integrate multiple host information for constructing the host-host similarity network"
- [results] The AUPR of ILMF-VH under five-fold cross-validation is maximized at a factorization factor k of 100, where the average AUPR reaches 0.6305.: [evidence span withheld — anchor status: not_found]
- [readme] (3) choose the optimal parameters for a method python PyDTI.py --method="nrlmf" --dataset="nr" --cvs=1 --specify-arg=0: "(3) choose the optimal parameters for a method

		python PyDTI.py --method="nrlmf" --dataset="nr" --cvs=1 --specify-arg=0"
