---
name: state-space-model-variational-inference
description: Use when you have a longitudinal OTU table (taxa by samples with sequence
  IDs and sampling times) of noisy sequencing counts and need denoised relative abundances
  plus posterior probabilities that observed zeros are sampling zeros rather than
  absences.
when_to_use_negative:
- Ultra-sparse taxa (>90% missing entries) dominate the table without pre-filtering
  — the README recommends removing them first because little information exists in
  their trajectory
- The goal is cross-sectional (single time point) compositional inference — LUMINATE
  is designed for longitudinal count data with temporal dependence between states
- A comparison baseline requires MALLARD's model in the Silverman et al. (2019) framework
  — the article notes no public implementation of MALLARD's in that framework exists
edam_operation: http://edamontology.org/operation_0306
tools:
- name: LUMINATE
  role: Primary implementation of the state-space variational inference estimator
    (estimate, train, predict, plot commands)
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Numerical backend for state-space computations such as finite-difference estimation
    of trajectories
- name: SciPy
  role: RK45 numerical integration used to forward-simulate ground-truth trajectories
    in the validation simulations
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:814d75cef08474a7030accfc499a794b152c35864d9529f537b9feeb8bfc7449
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - NumPy
  - RK45
  - SciPy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# state-space-model-variational-inference

## Summary

Run LUMINATE's state-space variational inference to reconstruct latent relative-abundance trajectories from longitudinal sequencing counts and to distinguish biological zeros (taxon absent) from technical zeros (taxon below detection threshold). The approach reformulates posterior inference in a state-space model as a structured optimization problem, making it orders of magnitude faster than TGP-CODA or MALLARD with similar or better accuracy.

## When to use

Use when you have a longitudinal OTU table (taxa by samples with sequence IDs and sampling times) of noisy sequencing counts and need denoised relative abundances plus posterior probabilities that observed zeros are sampling zeros rather than absences.

## When NOT to use

- Ultra-sparse taxa (>90% missing entries) dominate the table without pre-filtering — the README recommends removing them first because little information exists in their trajectory
- The goal is cross-sectional (single time point) compositional inference — LUMINATE is designed for longitudinal count data with temporal dependence between states
- A comparison baseline requires MALLARD's model in the Silverman et al. (2019) framework — the article notes no public implementation of MALLARD's in that framework exists

## Inputs

- OTU table CSV (taxa by samples) with sequence IDs in row 1 and sampling times in row 2, e.g. datasets/bucci2016mdsine/cdiff-counts.csv
- Optional external perturbation events CSV with columns sequenceID,eventID,startDay,endDay,magnitude(optional), e.g. cdiff-events.csv

## Outputs

- cdiff-counts-est.csv: estimated relative abundances
- cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities that observed zeros are sampling zeros rather than absences
- P.pkl: temporary result used for downstream analysis (e.g. cLV parameter training)
- Stacked bar plots of estimated relative abundances per sequence (top 19 taxa uniquely colored, remainder aggregated)

## How to apply

Load the OTU table (e.g. datasets/bucci2016mdsine/cdiff-counts.csv) where row 1 holds sequence IDs and row 2 sampling times, and optionally supply external perturbations via an events CSV with columns sequenceID,eventID,startDay,endDay,magnitude. As pre-processing, filter out ultra-sparse taxa (>90% missing entries) since their trajectories carry little information. Run LUMINATE's estimate program, which performs coordinate ascent variational inference over the state-space model: update q(x_1:T) in closed form using the block-tridiagonal precision inverse exploiting the O(TD^3) banded structure, update q(w) for the hidden Markov zero indicators with the forward-backward equations, and optimize the variational means h_1:T for q(z) with a conjugate gradient algorithm, iterating until m_1:T, h_1:T, and parameters converge. Collect the three outputs (estimated relative abundances, nonzero posterior probabilities, P.pkl) and plot the estimated abundances with the events file to verify the trajectories are smoother than the raw counts.

## Related tools

- **LUMINATE** (Primary implementation of the state-space variational inference estimator (estimate, train, predict, plot commands)) — https://github.com/tyjo/luminate
- **NumPy** (Numerical backend for state-space computations such as finite-difference estimation of trajectories)
- **SciPy** (RK45 numerical integration used to forward-simulate ground-truth trajectories in the validation simulations)

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- The three estimate output files exist: cdiff-counts-est.csv, cdiff-counts-nonzero-posterior-prob.csv, and P.pkl
- Mean r^2 per taxon across samples between ground-truth and estimated trajectories matches the paper's evaluation criterion
- On simulated biological-zero data (4 taxa, 30 days, gLV with one taxon extinct), posterior probabilities q(w_t^d = 0) separate true zeros from technical zeros with mean AUC-ROC ~0.91 (std 0.04) across 10 replicates
- Plotted estimated relative abundances are visibly smoother than the raw noisy counts per sequence
- The coordinate ascent iteration terminates with converged m_1:T, h_1:T, and model parameters A_1:D, Q, r_1:D-1

## Limitations

- The biological zero detection framework does not yet include external perturbations such as antibiotics (noted as future work)
- Downstream analysis of biological interaction networks among taxa is not yet implemented
- Training cLV parameters with LUMINATE is considered experimental; for ultra-sparse taxa (>90% missing entries) there is little trajectory information
- TGP-CODA and MALLARD baselines have difficulty scaling with sample size and taxa, and no public implementation of MALLARD's in the Silverman et al. (2019) framework exists

## Evidence

- [abstract] We demonstrate that LUMINATE is orders of magnitude faster than current approaches, with better or similar accuracy.: "We demonstrate that LUMINATE is orders of magnitude faster than current approaches, with better or similar accuracy."
- [abstract] Biological vs technical zero discrimination: "We further show that LUMINATE can accurately distinguish biological zeros, when a taxon is absent from the community, from technical zeros, when a taxon is below the detection threshold."
- [methods] Evaluation metric: "To evaluate model performance, we computed the mean r2, per taxon across samples between the ground truth and estimated trajectories"
- [readme] Estimate outputs: "cdiff-counts-nonzero-posterior-prob.csv`: posterior probabilities of sampling zeros (i.e. a taxon is below the detection threshold as opposed to absent from the community)"
- [readme] Sparsity pre-processing: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
- [readme] OTU table input format: "Each program takes as input an OTU Table (taxa by samples) with the following structure: *  The first row gives IDs for each longitudinal sequence. *  The second row gives sample times."
