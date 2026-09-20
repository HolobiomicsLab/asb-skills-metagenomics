---
name: additive-log-ratio-compositional-transformation
description: Use when longitudinal sequencing count or relative-abundance data are
  compositional and must be embedded in unconstrained ALR coordinates before modeling
  trajectories, computing SNR, or interpreting cLV parameters.
when_to_use_negative:
- Data are raw counts where an absolute-abundance or centered log-ratio transform
  is required instead, or the reference taxon x_D is unreliable/absent
- The taxon intended as the reference denominator is ultra-sparse (>90% missing entries)
  and should first be filtered out as a pre-processing step
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LUMINATE
  role: Implements the state-space model whose latent states and cLV parameters (A,
    g, B) are expressed in ALR coordinates
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Used for the finite-difference and SNR computations performed under the additive
    log-ratio transformation
- name: MGLM
  role: Estimates the overdispersion parameter alpha (a = 251.2809) on the C. diff
    dataset used in ALR-based noise simulations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:d5af7ff7799c89ae0302610341c1ece72b82d71d4e752cfeee06b99a1d2c3c44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - MGLM
  - NumPy
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# additive-log-ratio-compositional-transformation

## Summary

Applies the additive log-ratio (alr) transformation—log(x_i(t)/x_D(t)) against a reference taxon—to turn compositional relative abundances into unconstrained real-valued coordinates, as used in LUMINATE's state-space model for longitudinal count data.

## When to use

Use when longitudinal sequencing count or relative-abundance data are compositional and must be embedded in unconstrained ALR coordinates before modeling trajectories, computing SNR, or interpreting cLV parameters.

## When NOT to use

- Data are raw counts where an absolute-abundance or centered log-ratio transform is required instead, or the reference taxon x_D is unreliable/absent
- The taxon intended as the reference denominator is ultra-sparse (>90% missing entries) and should first be filtered out as a pre-processing step

## Inputs

- OTU table (taxa by samples) as CSV, e.g. datasets/bucci2016mdsine/cdiff-counts.csv, with sequence IDs in row 1 and sampling times in row 2
- Optional external perturbation table (.csv with columns sequenceID,eventID,startDay,endDay,magnitude)
- Ground-truth relative abundance trajectories (for SNR evaluation)

## Outputs

- ALR-coordinate latent states x_t (estimated relative abundances, e.g. cdiff-counts-est.csv)
- Model parameter matrices A, g, B with rows as ALR coordinates plus bootstrap p-value matrices A_pval, g_pval, B_pval
- SNR values computed under the ALR transformation

## How to apply

Represent each taxon's abundance relative to a reference taxon (x_D) by computing alr(x_i(t)) = log(x_i(t)/x_D(t)), which maps the D-simplex to R^(D-1) so Gaussian state-space dynamics can be applied. In LUMINATE, the latent states x_t live in ALR space and are evolved as Gaussian random walks scaled by inter-observation time Δt; the final model parameters A (taxa interactions), g (growth rates), and B (external effects) are reported with rows in ALR coordinates. For benchmarking, the same transformation defines the SNR: compute log-normal Poisson sequencing depth noise under alr coordinates. Zero handling is done in parallel via hidden Markov indicators w_t distinguishing biological from technical zeros rather than naive pseudo-counts (pseudo-counts are only an option via --use-pseudo-counts). Filter ultra-sparse taxa (>90% missing entries) before transformation, since their trajectories carry too little information.

## Related tools

- **LUMINATE** (Implements the state-space model whose latent states and cLV parameters (A, g, B) are expressed in ALR coordinates) — https://github.com/tyjo/luminate
- **NumPy** (Used for the finite-difference and SNR computations performed under the additive log-ratio transformation)
- **MGLM** (Estimates the overdispersion parameter alpha (a = 251.2809) on the C. diff dataset used in ALR-based noise simulations)

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- Check that transformed coordinates sum to zero in ALR space relative to the chosen reference taxon x_D (i.e., D taxa yield D-1 coordinates)
- Estimate outputs (cdiff-counts-est.csv) should have relative abundances that re-normalize to the simplex per time point
- Judge trajectory accuracy with the paper's criterion: mean r^2 per taxon across samples between ground truth and estimated trajectories
- Bootstrap p-values in A_pval, g_pval, B_pval must be >= 1/b (minimum possible p-value), where b is the bootstrap count (e.g., -b 200)
- SNR under alr should decrease as log-normal Poisson sequencing depth noise variance increases

## Limitations

- Ultra-sparse taxa (>90% missing entries) carry little trajectory information and are recommended for removal as a pre-processing step before transformation
- Estimates of cLV parameters from the transformed states are considered experimental by the LUMINATE README
- Zero values must be handled by the biological/technical zero HMM framework; pseudo-counts (--use-pseudo-counts) are a crude alternative

## Evidence

- [methods] ALR defines coordinates for SNR: [evidence span withheld — anchor status: not_found]
- [readme] Parameters reported in ALR coordinates: "The rows of each matrix, `A, g, B`, are coordinates under the additive log ratio transformation."
- [readme] Sparse-taxa filtering rationale: "The information available to reconstruct a trajectory is proportional to the amount of nonmissing (nonzero) data."
- [methods] Noise model parameter estimated on C. diff: [evidence span withheld — anchor status: not_found]
