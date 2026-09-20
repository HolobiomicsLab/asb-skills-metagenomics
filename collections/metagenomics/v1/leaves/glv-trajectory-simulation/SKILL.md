---
name: glv-trajectory-simulation
description: Use when you need synthetic ground-truth longitudinal trajectories of
  taxa abundances (e.g., 16 taxa over 56 days or 17 taxa over 65 days) on which to
  overlay sequencing noise for benchmarking reconstruction methods.
when_to_use_negative:
- The input is already denoised relative abundances rather than raw noisy longitudinal
  count data needing a synthetic benchmark
- The community dynamics are known to violate gLV assumptions (e.g., strong non-equilibrium
  perturbation dynamics such as antibiotic pulses not modeled in the gLV terms)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: SciPy
  role: Provides the RK45 integrator used to forward-simulate gLV trajectories from
    each mouse's initial conditions
- name: RK45
  role: Runge-Kutta 5(4) numerical integration method used for the forward simulation
- name: NumPy
  role: Used for finite-difference estimation of f0d(t) when learning gLV parameters
    prior to simulation
- name: MGLM
  role: Estimates Dirichlet-Multinomial dispersion (alpha = 251.2809 on the C. diff
    dataset) used for the alternative noise simulation on top of simulated trajectories
- name: LUMINATE
  role: Downstream consumer of the simulated noisy count datasets for benchmarking
    trajectory reconstruction
  repo: https://github.com/tyjo/luminate
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:0c92f1c036509557445a2002f590d636a69d4e2e7291721a8a6c0fab9a6c0ced
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - MALLARD
  - MGLM
  - NumPy
  - PyStan
  - RK45
  - RStan
  - SciPy
  - TGP-CODA
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# glv-trajectory-simulation

## Summary

Simulate ground-truth microbial community trajectories by fitting generalized Lotka-Volterra (gLV) parameters to longitudinal count data (e.g., Bucci et al. C. diff and Diet mouse datasets) and forward-integrating trajectories per mouse with the Runge-Kutta 5(4) RK45 integrator from SciPy.

## When to use

Use when you need synthetic ground-truth longitudinal trajectories of taxa abundances (e.g., 16 taxa over 56 days or 17 taxa over 65 days) on which to overlay sequencing noise for benchmarking reconstruction methods.

## When NOT to use

- The input is already denoised relative abundances rather than raw noisy longitudinal count data needing a synthetic benchmark
- The community dynamics are known to violate gLV assumptions (e.g., strong non-equilibrium perturbation dynamics such as antibiotic pulses not modeled in the gLV terms)

## Inputs

- Longitudinal concentration/count datasets from Bucci et al. 2016: C. diff dataset (5 gnotobiotic mice, 16 species) and Diet dataset (7 germ-free mice, 17 Clostridia strains) from bitbucket.org/MDSINE/mdsine
- OTU table (taxa by samples) with sequence IDs, sample times, and taxon names in the first column
- gLV model parameters learned by discretized least squares with elastic net regularization
- Initial conditions (relative abundances) for each mouse

## Outputs

- 5 forward-simulated C. diff gLV trajectories (16 taxa, ~26 average time points over 56 days)
- 7 forward-simulated Diet gLV trajectories (17 taxa, ~47 time points over 65 days)
- Simulated sequencing count tables overlaid on ground-truth trajectories (log-normal Poisson or Dirichlet-Multinomial noise)

## How to apply

Download the C. diff and Diet longitudinal concentration/count datasets from bitbucket.org/MDSINE/mdsine, fit the gLV Equation 3 by discretizing it and performing least squares with elastic net regularization. Forward-simulate one trajectory per mouse from each mouse's initial conditions using the Runge-Kutta 5(4) RK45 integrator from SciPy, yielding 5 C. diff trajectories (16 taxa, ~26 time points over 56 days) and 7 Diet trajectories (17 taxa, ~47 time points over 65 days). Finally, simulate sequencing counts on top of each ground-truth trajectory (e.g., following the Silverman et al. framework with log-normal Poisson depth, or Dirichlet-Multinomial noise with dispersion estimated via MGLM) so the simulations can serve as benchmarks with known truth.

## Related tools

- **SciPy** (Provides the RK45 integrator used to forward-simulate gLV trajectories from each mouse's initial conditions)
- **RK45** (Runge-Kutta 5(4) numerical integration method used for the forward simulation)
- **NumPy** (Used for finite-difference estimation of f0d(t) when learning gLV parameters prior to simulation)
- **MGLM** (Estimates Dirichlet-Multinomial dispersion (alpha = 251.2809 on the C. diff dataset) used for the alternative noise simulation on top of simulated trajectories)
- **LUMINATE** (Downstream consumer of the simulated noisy count datasets for benchmarking trajectory reconstruction) — https://github.com/tyjo/luminate

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- Simulated trajectory dimensions match the paper's setup: 5 C. diff trajectories with 16 taxa over 56 days and 7 Diet trajectories with 17 taxa over ~65 days
- gLV parameter fit uses discretized least squares with elastic net regularization, reproducing the Stein et al. 2013 approach
- Sequencing noise simulation follows the Silverman et al. framework: SNR computed as Tr(Q)/Tr(R) under the additive log-ratio transformation, log-normal Poisson sequencing depth with log M_t ~ N(log 10000, 0.5)
- Dirichlet-Multinomial simulations use dispersion alpha = 251.2809 estimated by MGLM on post-convergence C. diff observations, tested at sequencing depths N = 5000, 10000, 20000
- Ground-truth trajectories are reproducible from each mouse's initial conditions via RK45 integration

## Limitations

- For ultra-sparse taxa (>90% missing entries) there is little information about the trajectory over time; such taxa are recommended to be filtered out as a pre-processing step
- LUMINATE's cLV parameter training on simulated/estimated trajectories is considered experimental, so downstream dynamical-system fitting from simulations carries that caveat

## Evidence

- [methods] Fit gLV by discretization and least squares with elastic net regularization: [evidence span withheld — anchor status: not_found]
- [methods] Forward-simulate trajectories with RK45: [evidence span withheld — anchor status: not_found]
- [methods] Simulate sequencing counts under varying noise levels: "We simulated sequencing counts on top of each ground truth trajectory under varying levels of sequencing noise, following the framework of Silverman et al. (2018)"
- [methods] Datasets used for gLV fitting: "i) a dataset of 5 gnotobiotic mice colonized with a bacterial mixture of 16 species (the C. diff dataset)"
- [readme] Filter ultra-sparse taxa before simulation-based benchmarking: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
