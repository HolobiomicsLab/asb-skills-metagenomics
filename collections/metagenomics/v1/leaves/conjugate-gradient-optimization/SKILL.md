---
name: conjugate-gradient-optimization
description: Use when estimating latent relative abundances from longitudinal microbiome
  count data (OTU tables such as cdiff-counts.csv) via LUMINATE's coordinate ascent
  VI, specifically when solving the inner optimization for noisy realizations z_1:T.
when_to_use_negative:
- The data are not longitudinal count data with sampling times — LUMINATE's state-space
  formulation assumes taxa-by-samples counts ordered over time (e.g. 16S longitudinal
  mouse studies).
- Ultra-sparse taxa (>90% missing entries) were not filtered — the authors recommend
  removing these as a pre-processing step because there is little information to reconstruct
  their trajectory over time.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LUMINATE
  role: Implements the state-space variational inference in which conjugate gradient
    optimizes the variational means h_1:T for q(z_1:T) as part of the estimate command
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Numerical backend for finite differences and array operations in the LUMINATE
    pipeline
- name: SciPy
  role: Provides numerical routines used alongside LUMINATE (e.g. RK45 integration
    for simulation-based validation)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:e9241d1f2e494fdfa46fbdabb76e4b6457d1ea5f2a151643b23588e33c7e3697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - NumPy
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# conjugate-gradient-optimization

## Summary

Iteratively optimize the variational means h_1:T for q(z_1:T) in LUMINATE's state-space model using a conjugate gradient algorithm, as one step of the coordinate ascent variational inference loop that reconstructs relative abundances and detects biological versus technical zeros in longitudinal sequencing counts.

## When to use

Use when estimating latent relative abundances from longitudinal microbiome count data (OTU tables such as cdiff-counts.csv) via LUMINATE's coordinate ascent VI, specifically when solving the inner optimization for noisy realizations z_1:T.

## When NOT to use

- The data are not longitudinal count data with sampling times — LUMINATE's state-space formulation assumes taxa-by-samples counts ordered over time (e.g. 16S longitudinal mouse studies).
- Ultra-sparse taxa (>90% missing entries) were not filtered — the authors recommend removing these as a pre-processing step because there is little information to reconstruct their trajectory over time.

## Inputs

- OTU table (taxa by samples) with sequence IDs in row 1 and sampling times in row 2, e.g. datasets/bucci2016mdsine/cdiff-counts.csv
- Optional external perturbations CSV with columns sequenceID,eventID,startDay,endDay,magnitude

## Outputs

- cdiff-counts-est.csv: estimated relative abundances
- cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities of sampling zeros (technical zeros vs biological zeros)
- P.pkl: temporary result used for downstream analysis (e.g. cLV training)

## How to apply

Within LUMINATE's coordinate ascent variational inference, alternate between closed-form updates of q(x_1:T) via the block-tridiagonal precision inverse (Equations 1 and 2, exploiting O(TD^3) banded structure) and numerical optimization of the variational means h_1:T for q(z_1:T) with a conjugate gradient algorithm. The conjugate gradient step handles the noisy realizations z_t that link latent additive-log-ratio states x_t to observed sequencing counts, while q(w_d_1:T) for the hidden Markov zero indicators is updated separately with forward-backward equations (O(4T) per taxon). After each sweep, update model parameters A_1:D, Q, and r_1:D-1 in closed form. Iterate the whole loop until convergence of m_1:T, h_1:T, and the parameters. Optionally pre-filter ultra-sparse taxa (>90% missing entries) before running, since such taxa carry too little information to reconstruct their trajectory.

## Related tools

- **LUMINATE** (Implements the state-space variational inference in which conjugate gradient optimizes the variational means h_1:T for q(z_1:T) as part of the estimate command) — https://github.com/tyjo/luminate
- **NumPy** (Numerical backend for finite differences and array operations in the LUMINATE pipeline)
- **SciPy** (Provides numerical routines used alongside LUMINATE (e.g. RK45 integration for simulation-based validation))

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- The coordinate ascent loop terminates with convergence of m_1:T, h_1:T, and model parameters A_1:D, Q, r_1:D-1.
- Running `estimate` produces all three expected files: cdiff-counts-est.csv, cdiff-counts-nonzero-posterior-prob.csv, and P.pkl.
- Estimated relative abundances are judged by the paper's criterion of mean r^2 per taxon across samples against ground-truth trajectories.
- Posterior probabilities of sampling zeros accurately distinguish technical zeros (taxon below detection threshold) from biological zeros (taxon absent from the community).
- Plotting the estimated relative abundances after estimation should yield smoothed trajectories over time without errors in the stacked bar plot.

## Limitations

- Estimates of cLV parameters with LUMINATE's train command are considered experimental by the authors.
- For ultra-sparse taxa (>90% missing entries), little information exists to reconstruct trajectories, so they should be filtered out beforehand.
- Existing state-space alternatives (TGP-CODA, MALLARD) have difficulty scaling with sample size and taxa; conversely, no public implementation of MALLARD in the Silverman et al. (2019) framework exists for direct comparison.
- The biological zero detection framework does not yet include external perturbations such as antibiotics, and downstream interaction network learning is not yet implemented.

## Evidence

- [methods] Coordinate ascent VI combines closed-form state updates with CG optimization: [evidence span withheld — anchor status: not_found]
- [methods] Conjugate gradient optimizes variational means for noisy realizations: [evidence span withheld — anchor status: not_found]
- [methods] Sparsity filter precedes inference: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
- [readme] Estimate command outputs the skill's target artifacts: "cdiff-counts-est.csv : the estimated relative abundances"
- [readme] Zero posterior probabilities as a concrete output artifact: "cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities of sampling zeros"
