---
name: top-k-consensus-host-prediction
description: Use when a scoring model (e.g., kernelized logistic matrix factorization
  on a virus-host heterogeneous network) outputs ranked host predictions per virus
  and you want a more robust species-level host call than the single top-ranked prediction
  provides.
when_to_use_negative:
- The question requires a single definitive host per virus (e.g., strain-level attribution),
  since consensus blurs ranks into a top-k call
- The prediction model already reports calibrated probabilities or a validated single-host
  decision rule rather than a score ranking
- Evaluation is restricted to a metric other than species-level accuracy of the assigned
  host (e.g., AUROC over all pairs), where top-k aggregation is not the target
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3315
tools:
- name: ILMF-VH
  role: Produces the ranked virus-host association scores whose top-5 ranks are aggregated
    into the consensus host prediction
  repo: github.com/liudan111/ILMF-VH
- name: PyDTI
  role: Baseline library (NRLMF and related methods) whose code base underpins ILMF-VH's
    scoring and cross-validation execution
  repo: github.com/stephenliu0423/PyDTI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3082-0
    title: Predicting virus-host association by Kernelized logistic matrix factorization
      and similarity network fusion
  source_content_hash: sha256:fcc78155ae37dfc55d23f546cca06dd15c6d3b9ca32ef43b66284af05c6a5af7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ILMF-VH
  - Jie Ren's tools
  - PyDTI
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# top-k-consensus-host-prediction

## Summary

Boosts ILMF-VH virus host prediction accuracy by aggregating the top-ranked predicted hosts into a top-5 consensus call instead of reporting only the single highest-scoring host. On the Edwards et al. benchmark (820 viruses, 2699 hosts), consensus lifted species-level accuracy from 58.90% (top-1) to 63.66%.

## When to use

Use when a scoring model (e.g., kernelized logistic matrix factorization on a virus-host heterogeneous network) outputs ranked host predictions per virus and you want a more robust species-level host call than the single top-ranked prediction provides.

## When NOT to use

- The question requires a single definitive host per virus (e.g., strain-level attribution), since consensus blurs ranks into a top-k call
- The prediction model already reports calibrated probabilities or a validated single-host decision rule rather than a score ranking
- Evaluation is restricted to a metric other than species-level accuracy of the assigned host (e.g., AUROC over all pairs), where top-k aggregation is not the target

## Inputs

- Predicted virus-host association score matrix from ILMF-VH for all benchmark viruses (820 viruses) against all hosts (2699 hosts)
- Ground-truth species-level host labels from the Edwards et al. benchmark virus-host data (RefSeq, downloaded 7/25/2015)

## Outputs

- Species-level host prediction accuracy of the top-ranked (direct) prediction, e.g. 58.90% on the benchmark
- Species-level host prediction accuracy of the top-5 consensus method, e.g. 63.66% on the benchmark
- Results table (e.g., benchmark_results.csv) comparing direct vs consensus accuracies

## How to apply

After running ILMF-VH (kernelized logistic matrix factorization with similarity network fusion) to obtain predicted association scores between each virus and all 2699 hosts, rank hosts by predicted score for each virus. Instead of taking only the top-1 host, derive a consensus host from the top-5 ranked predictions and evaluate species-level accuracy under five-fold cross validation. The rationale is that the correct host frequently appears within the top few ranks even when it is not the highest-scoring one, so aggregating top-ranked candidates recovers it more often (63.66% vs 58.90% in the paper's Figure 2 comparison). Record both top-1 and top-5 consensus accuracies (e.g., to benchmark_results.csv) so the consensus gain can be quantified against the reported baseline.

## Related tools

- **ILMF-VH** (Produces the ranked virus-host association scores whose top-5 ranks are aggregated into the consensus host prediction) — github.com/liudan111/ILMF-VH
- **PyDTI** (Baseline library (NRLMF and related methods) whose code base underpins ILMF-VH's scoring and cross-validation execution) — github.com/stephenliu0423/PyDTI

## Examples

```
python PyDTI.py --method="nrlmf" --dataset="dataset II" --cvs=1 --specify-arg=1 --predict-num=5
```

## Evaluation signals

- Top-1 species-level accuracy on the 820-virus benchmark should reproduce ~58.90% under five-fold cross validation
- Top-5 consensus accuracy should exceed the top-1 accuracy and reproduce ~63.66% as reported in Figure 2
- Consensus must be computed from the same score matrix used for the direct prediction, so both accuracies derive from identical cross-validation folds
- Predicted scores are defined for all 820 viruses × 2699 hosts; every benchmark virus must receive a ranked host list

## Limitations

- Consensus relies on the true host appearing within the top ranks; it does not improve cases where the correct host scores below the top-5
- Species-level accuracy of purely top-1 prediction remains below 60%; consensus narrows but does not eliminate the gap with genus-level performance
- Sequence homology approaches depend on whether hosts of the query virus exist in the host genome database, limiting comparability of baselines on novel viruses
- Implementation is Python 2.7.9 only (per README), which may complicate direct reuse

## Evidence

- [results] Top-5 consensus accuracy claim: [evidence span withheld — anchor status: not_found]
- [intro] Cross-validation and benchmark comparison protocol: "verified that ILMF-VH obtained best performance compared with recent five network-based methods under five-fold cross validation."
- [intro] Goal of improving species-level accuracy over prior <40% baselines: "This method achieves good results in host prediction accuracy at the genus level, but less than 40% at the species level."
- [readme] Benchmark dataset composition (dataset II: 2699 hosts, 820 viruses): "datasets/dataset II including 2699 hosts and 820 viruses which is downloaded from RefSeq on 7/25/2015[1]."
