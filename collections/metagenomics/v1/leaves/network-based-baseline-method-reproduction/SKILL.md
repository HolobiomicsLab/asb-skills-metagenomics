---
name: network-based-baseline-method-reproduction
description: Use when a new network-based association predictor must be benchmarked
  against published baselines on a known association matrix with precomputed similarity
  networks, and reported AUC/AUPR values must be reproduced to validate the comparison.
when_to_use_negative:
- The goal is de novo host prediction for new viruses rather than benchmarking against
  published baselines on a labeled association matrix.
- The association data are drug-target pairs (PyDTI's native nr, gpcr, ic, e datasets)
  rather than virus-host associations, unless the same CV protocol is being deliberately
  ported.
- No precomputed similarity networks are available and the task does not include constructing
  virus dissimilarity (Jie Ren's d2* tools) or host SNF networks first.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0084
tools:
- name: PyDTI
  role: Provides the baseline network-based methods (KBMF2K, BLM-NII, NetLapRLS, CMF)
    run under five-fold cross validation via PyDTI.py with --method, --dataset, and
    --cvs parameters
  repo: github.com/stephenliu0423/PyDTI
- name: ILMF-VH code
  role: Hosts nrlmf.py and PyDTI.py adapted for virus-host prediction, plus datasets
    I and II and the onf measures; KBMF/NetLapRLS/BLM-NII/CMF refer back to Yong et
    al.'s PyDTI
  repo: github.com/liudan111/ILMF-VH
- name: Jie Ren's d2* oligonucleotide frequency tools
  role: Compute virus-virus dissimilarity which must be transformed to similarity
    before building the virus network used by the baselines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:bad3ea84d4e9f8efcc94a74d02b82768bdd9369584a19bfaadb1e1eea93f60fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ILMF-VH code
  - Jie Ren's d2* oligonucleotide frequency tools
  - PyDTI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# network-based-baseline-method-reproduction

## Summary

Reproduces benchmark AUC/AUPR results for five recent network-based virus-host prediction baselines (LMFH-VH, KBMF2K, BLM-NII, NetLapRLS, CMF) under five-fold cross validation using the PyDTI framework, so a new method such as ILMF-VH can be fairly compared against Table 2 reference values.

## When to use

Use when a new network-based association predictor must be benchmarked against published baselines on a known association matrix with precomputed similarity networks, and reported AUC/AUPR values must be reproduced to validate the comparison.

## When NOT to use

- The goal is de novo host prediction for new viruses rather than benchmarking against published baselines on a labeled association matrix.
- The association data are drug-target pairs (PyDTI's native nr, gpcr, ic, e datasets) rather than virus-host associations, unless the same CV protocol is being deliberately ported.
- No precomputed similarity networks are available and the task does not include constructing virus dissimilarity (Jie Ren's d2* tools) or host SNF networks first.

## Inputs

- Known virus-host association matrix for dataset I (352 viruses x 71 hosts)
- Precomputed virus-virus similarity network derived from d2* oligonucleotide frequency dissimilarity (transformed to similarity)
- Precomputed host-host similarity network from similarity network fusion of oligonucleotide frequency similarity and Gaussian interaction profile kernel similarity
- PyDTI baseline implementations (LMFH-VH/NRLMF-style, KBMF2K, BLM-NII, NetLapRLS, CMF)
- Matlab KBMF2K code by M. Gonen with a kbmf.m predict wrapper (for KBMF2K only)

## Outputs

- Per-method mean AUC and AUPR under five-fold cross validation (e.g. cv_results_table.csv)
- Held-out prediction scores aggregated across folds for each method
- A verified comparison table matching the reported Table 2 values

## How to apply

Load the target dataset (e.g. dataset I: 352 viruses, 71 hosts) with its known virus-host association matrix and the precomputed virus-virus (oligonucleotide-frequency) and host-host (SNF-fused) similarity networks. For each of five folds, train each baseline — LMFH-VH, KBMF2K, BLM-NII, NetLapRLS, and CMF taken from the PyDTI framework of Yong Liu et al. — together with the proposed method on the remaining folds. KBMF2K additionally requires Matlab code from M. Gonen bridged via pymatbridge with a kbmf.m wrapper function. Aggregate held-out predictions across folds, compute AUC and AUPR per method, and write the mean values into a results table. Judgment of correctness: each baseline's reproduced AUC/AUPR must match the paper's Table 2 values (e.g. ILMF-VH AUC 0.9202, AUPR 0.6243), confirming the proposed method obtained best performance among the six methods.

## Related tools

- **PyDTI** (Provides the baseline network-based methods (KBMF2K, BLM-NII, NetLapRLS, CMF) run under five-fold cross validation via PyDTI.py with --method, --dataset, and --cvs parameters) — github.com/stephenliu0423/PyDTI
- **ILMF-VH code** (Hosts nrlmf.py and PyDTI.py adapted for virus-host prediction, plus datasets I and II and the onf measures; KBMF/NetLapRLS/BLM-NII/CMF refer back to Yong et al.'s PyDTI) — github.com/liudan111/ILMF-VH
- **Jie Ren's d2* oligonucleotide frequency tools** (Compute virus-virus dissimilarity which must be transformed to similarity before building the virus network used by the baselines)

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="nr" --cvs=1 --specify-arg=1 --method-opt="r=100"
```

## Evaluation signals

- Reproduced five-fold CV AUC/AUPR for ILMF-VH match the Table 2 reference values: AUC 0.9202, AUPR 0.6243 on the 352-virus dataset.
- Each baseline's mean AUC/AUPR matches its Table 2 value rather than only the proposed method's.
- ILMF-VH ranks first among the six methods, i.e. 'ILMF-VH obtained best performance compared with recent five network-based methods under five-fold cross validation'.
- All methods are evaluated on identical fold splits of the same known virus-host associations, so held-out prediction sets are comparable across methods.
- cv_results_table.csv is written with one mean AUC and AUPR row per method and no missing methods.

## Limitations

- KBMF2K requires external Matlab code and pymatbridge; PyDTI is implemented for Python 2.7.9, which may complicate modern environments.
- Jie Ren's tools output dissimilarity between viruses, so an explicit dissimilarity-to-similarity transformation step is required and is easy to omit.
- The comparison depends on hosts of query viruses existing in the reference genome database; homology-based coverage is limited for novel viruses.

## Evidence

- [intro] verified that ILMF-VH obtained best performance compared with recent five network-based methods under five-fold cross validation.: "verified that ILMF-VH obtained best performance compared with recent five network-based methods under five-fold cross validation."
- [readme] The codes of KBMF,NetLapRLS,BLM-NII ,CMF can refer to codes of Yong et al.: "The codes of KBMF,NetLapRLS,BLM-NII ,CMF can refer to codes of Yong et al.https://github.com/stephenliu0423/PyDTI"
- [readme] Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity.: "Note: Jie Ren's tools is used to get dissimilarity between viruses,you should transform to similarity."
- [readme] datasets/dataset I including 71 hosts and 352 viruses which only one specific host genome appeared in the prokaryotic genome database at NCBI: "datasets/dataset I including 71 hosts and 352 viruses which only one specific host genome appeared in the prokaryotic genome database at NCBI[2]."
- [readme] Note that pymatbridge is only required by KBMF2K.: [evidence span withheld — anchor status: external_doc]
