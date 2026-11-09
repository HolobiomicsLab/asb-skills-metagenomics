---
name: hidden-markov-model-forward-backward-inference
description: Use when longitudinal sequencing count data contain observed zeros and
  you need posterior probabilities distinguishing biological zeros (extinct taxa)
  from technical zeros (below detection threshold), e.g. in a state-space/variational
  inference model of community composition.
when_to_use_negative:
- The data have no observed zeros — the HMM zero-indicator layer is unnecessary and
  standard trajectory estimation suffices
- A taxon is ultra-sparse (>90% missing entries), which the paper recommends filtering
  out as a pre-processing step since there is little information about its trajectory
  over time
- The task is cross-sectional (no longitudinal structure), since the forward-backward
  updates exploit temporal dependencies between adjacent time points
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LUMINATE
  role: Method whose variational inference uses forward-backward HMM updates of q(w_d_1:T)
    for biological vs technical zero detection
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Loading the OTU table and implementing the linear-algebra components around
    the HMM updates
- name: SciPy
  role: Numerical routines (RK45) used to generate the gLV simulation data on which
    zero discrimination is evaluated
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:b5ac64537d7a2d08017ba826337db83ab31d8fc50f19ce4cd879b5c3d47d7f8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - NumPy
  - RK45
  - SciPy
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hidden-markov-model-forward-backward-inference

## Summary

Applies forward-backward dynamic programming for hidden Markov models, as used in LUMINATE's variational inference, to compute posteriors over per-taxon, per-timepoint zero indicators w_t^d that discriminate biological zeros (taxon absent from the community) from technical zeros (taxon below detection threshold).

## When to use

Use when longitudinal sequencing count data contain observed zeros and you need posterior probabilities distinguishing biological zeros (extinct taxa) from technical zeros (below detection threshold), e.g. in a state-space/variational inference model of community composition.

## When NOT to use

- The data have no observed zeros — the HMM zero-indicator layer is unnecessary and standard trajectory estimation suffices
- A taxon is ultra-sparse (>90% missing entries), which the paper recommends filtering out as a pre-processing step since there is little information about its trajectory over time
- The task is cross-sectional (no longitudinal structure), since the forward-backward updates exploit temporal dependencies between adjacent time points

## Inputs

- OTU table (taxa by samples) with sequence IDs in row 1 and sampling times in row 2, e.g. datasets/bucci2016mdsine/cdiff-counts.csv
- Latent-state variables w_t^d (hidden Markov zero indicators) and noisy realizations z_t from the LUMINATE state-space model
- Simulated datasets with known biological vs technical zeros (from simulate_glv_with_zeros.py, 4 taxa over 30 days under gLV)

## Outputs

- Posterior probabilities q(w_t^d = 0) for each observed zero (biological-zero posterior probabilities), e.g. cdiff-counts-nonzero-posterior-prob.csv
- Aggregated per-zero posterior probabilities across replicates, e.g. biological_zero_posterior_probabilities.csv

## How to apply

Model the zero indicators w_t^d for each taxon d as hidden Markov states over the longitudinal time points, so that adjacent time points share information about whether a taxon is extinct or merely below the detection threshold. Inside LUMINATE's coordinate ascent variational inference, update q(w_d_1:T) and its pairwise terms using the forward-backward equations, which runs in O(4T) per taxon. The forward-backward updates yield the posterior probability that each observed zero is a biological zero, q(w_t^d = 0), which is aggregated across replicates (e.g. 10 simulated gLV datasets of 10 longitudinal samples each, generated via simulate_glv_with_zeros.py). Validate by checking that posterior biological-zero probabilities separate true zeros from technical zeros, e.g. via AUC-ROC (the paper reports mean AUC-ROC = 0.91, std = 0.04).

## Related tools

- **LUMINATE** (Method whose variational inference uses forward-backward HMM updates of q(w_d_1:T) for biological vs technical zero detection) — https://github.com/tyjo/luminate
- **NumPy** (Loading the OTU table and implementing the linear-algebra components around the HMM updates)
- **SciPy** (Numerical routines (RK45) used to generate the gLV simulation data on which zero discrimination is evaluated)

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- Posterior biological-zero probabilities separate true zeros (extinct taxon) from technical zeros (taxon near detection threshold); the paper reports mean AUC-ROC = 0.91 (std = 0.04) across 10 simulated replicates
- Running the estimate command terminates and writes cdiff-counts-nonzero-posterior-prob.csv with posterior probabilities of sampling zeros
- Posterior probabilities are bounded in [0, 1] and aggregated per zero across all 10 simulated datasets
- On ground-truth simulations, posterior probability q(w_t^d = 0) is high for the extinct taxon and low for the taxon held near the detection threshold

## Limitations

- The biological zero detection framework does not yet include external perturbations such as antibiotics (noted as future work)
- Ultra-sparse taxa (>90% missing entries) provide little information about trajectory over time and should be filtered before inference
- Training cLV parameters with LUMINATE is considered experimental per the README

## Evidence

- [other] Update q(w_d_1:T) and its pairwise terms using the forward-backward equations for hidden Markov models (O(4T) per taxon).: [evidence span withheld — anchor status: not_found]
- [abstract] We further show that LUMINATE can accurately distinguish biological zeros, when a taxon is absent from the community, from technical zeros, when a taxon is below the detection threshold.: "We further show that LUMINATE can accurately distinguish biological zeros, when a taxon is absent from the community, from technical zeros, when a taxon is below the detection threshold."
- [methods] We trained our model across 10 datasets of 10 longitudinal samples each, and for each observed zero computed the posterior probability that it was a biological zero: [evidence span withheld — anchor status: not_found]
- [readme] cdiff-counts-nonzero-posterior-prob.csv : posterior probabilities of sampling zeros (i.e. a taxon is below the detection threshold as opposed to absent from the community): "cdiff-counts-nonzero-posterior-prob.csv`: posterior probabilities of sampling zeros (i.e. a taxon is below the detection threshold as opposed to absent from the community)"
- [readme] For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step.: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
