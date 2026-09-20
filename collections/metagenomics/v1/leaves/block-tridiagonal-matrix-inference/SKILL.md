---
name: block-tridiagonal-matrix-inference
description: Use when reconstructing latent relative-abundance trajectories from longitudinal
  OTU/count tables (taxa by samples) where sequencing noise causes technical zeros,
  and you need efficient posterior inference plus biological-vs-technical zero discrimination.
when_to_use_negative:
- Your data are cross-sectional or otherwise non-longitudinal — the model pools information
  across longitudinal samples via the state-space structure and cannot be applied
  to single-time-point counts.
- The OTU table contains ultra-sparse taxa (>90% missing entries); the README recommends
  filtering these out first, and reconstructing their trajectories yields little information.
- You need inference under external perturbation dynamics or full gLV parameter prediction
  — cLV training and prediction are flagged as experimental in the README.
tools:
- name: LUMINATE
  role: 'Implements the state-space variational inference: estimate, train, predict,
    and plot commands over longitudinal OTU tables'
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Array/linear-algebra backend for the banded precision computations and finite-difference
    estimates
- name: SciPy
  role: Numerical routines (e.g. RK45 forward simulation used to build the synthetic
    benchmarks the inference is validated on)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:468d41e4cbf9b0d945cfb51805dbeeaa9eeafc188343a5c62407dda418fe1fa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - NumPy
  - SciPy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# block-tridiagonal-matrix-inference

## Summary

LUMINATE's core inference engine: coordinate ascent variational inference for a state-space model of longitudinal microbiome counts, where q(x_1:T) is updated in closed form via a block-tridiagonal precision inverse exploiting O(TD^3) banded structure, q(w_d,1:T) for zero-type indicators is updated with HMM forward-backward recursions, and q(z_1:T) means are optimized by conjugate gradient.

## When to use

Use when reconstructing latent relative-abundance trajectories from longitudinal OTU/count tables (taxa by samples) where sequencing noise causes technical zeros, and you need efficient posterior inference plus biological-vs-technical zero discrimination.

## When NOT to use

- Your data are cross-sectional or otherwise non-longitudinal — the model pools information across longitudinal samples via the state-space structure and cannot be applied to single-time-point counts.
- The OTU table contains ultra-sparse taxa (>90% missing entries); the README recommends filtering these out first, and reconstructing their trajectories yields little information.
- You need inference under external perturbation dynamics or full gLV parameter prediction — cLV training and prediction are flagged as experimental in the README.

## Inputs

- OTU table (taxa by samples) CSV with sequence IDs in row 1, sample times in row 2, and taxon names in the first column (e.g. datasets/bucci2016mdsine/cdiff-counts.csv)
- Optional external perturbations CSV with columns sequenceID,eventID,startDay,endDay,magnitude (optional)

## Outputs

- cdiff-counts-est.csv: estimated relative abundances
- cdiff-counts-nonzero-posterior-prob.csv: posterior probabilities of sampling zeros (taxon below detection threshold vs absent)
- P.pkl: temporary result used for downstream cLV training

## How to apply

Load the OTU table (taxa by samples, sequence IDs in row 1, sample times in row 2, e.g. datasets/bucci2016mdsine/cdiff-counts.csv) and optionally filter ultra-sparse taxa (>90% missing entries) as pre-processing. Set up latent additive-log-ratio states x_t evolving as a Gaussian random walk scaled by inter-observation time Δt, noisy realizations z_t for sequencing counts, and hidden Markov indicators w_t distinguishing biological from technical zeros. Run coordinate ascent variational inference: update q(x_1:T) in closed form using the block-tridiagonal precision inverse (Equations 1 & 2) with its O(TD^3) banded structure, update q(w_d,1:T) via the forward-backward equations (O(4T) per taxon), and optimize the variational means h_1:T for q(z_1:T) with a conjugate gradient algorithm. Finally update model parameters A_1:D, Q, and r_1:D-1 in closed form and iterate until m_1:T, h_1:T, and the parameters converge; the --use-pseudo-counts flag trains on pseudo-counts instead of denoised estimates.

## Related tools

- **LUMINATE** (Implements the state-space variational inference: estimate, train, predict, and plot commands over longitudinal OTU tables) — https://github.com/tyjo/luminate
- **NumPy** (Array/linear-algebra backend for the banded precision computations and finite-difference estimates)
- **SciPy** (Numerical routines (e.g. RK45 forward simulation used to build the synthetic benchmarks the inference is validated on))

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- The run terminates and emits the three expected files: cdiff-counts-est.csv, cdiff-counts-nonzero-posterior-prob.csv, and P.pkl.
- Estimated trajectories achieve high mean r^2 per taxon across samples between ground truth and estimated trajectories (the paper's evaluation criterion).
- Posterior probabilities of sampling zeros correctly separate technical zeros (taxon below detection threshold) from biological zeros (taxon absent from the community).
- Iterates converge on m_1:T, h_1:T, and the parameters A_1:D, Q, r_1:D-1; performance stays comparable to or better than TGP-CODA/MALLARD while running orders of magnitude faster.
- Plotting cdiff-counts-est.csv (e.g. python main.py plot on the estimated table) shows smoothed trajectories rather than the noisy raw abundances.

## Limitations

- Ultra-sparse taxa (>90% missing entries) carry little information about their trajectory over time and should be filtered out as a pre-processing step.
- The biological zero detection framework does not yet include external perturbations such as antibiotics (noted as future work).
- Downstream analysis of biological interaction networks among taxa is not yet implemented (future work), and cLV parameter training/ prediction via LUMINATE is experimental.
- No public implementation of MALLARD's model in the Silverman et al. (2019) framework exists, limiting direct baseline comparisons.

## Evidence

- [intro] We propose LUMINATE (longitudinal microbiome inference and zero detection) as an accurate and efficient method to infer rela- tive abundances from microbial count data: "We propose LUMINATE (longitudinal microbiome inference and zero detection) as an accurate and efficient method to infer rela- tive abundances from microbial count data"
- [intro] variational inference reformulation: "First, using variational infer- ence, we reformulate the problem of posterior inference in a state-space model as an optimization problem with special structure."
- [methods] evaluation metric: "To evaluate model performance, we computed the mean r2, per taxon across samples between the ground truth and estimated trajectories"
- [readme] sparsity filter guidance: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
- [readme] estimate command outputs: "* `cdiff-counts-nonzero-posterior-prob.csv`: posterior probabilities of sampling zeros (i.e. a taxon is below the detection threshold as opposed to absent from the community)"
