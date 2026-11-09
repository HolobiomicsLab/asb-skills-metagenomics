---
name: longitudinal-count-noise-simulation
description: Use when you need synthetic longitudinal count benchmarks with known
  ground-truth trajectories to evaluate compositional denoising or trajectory-reconstruction
  methods such as LUMINATE, TGP-CODA, or MALLARD.
when_to_use_negative:
- Your goal is denoising real observed count data rather than generating synthetic
  benchmarks — use LUMINATE's 'estimate' command directly instead
- Your data lacks longitudinal structure or known per-mouse initial conditions, so
  gLV forward simulation cannot be grounded
- You need biological-zero discrimination benchmarks rather than plain trajectory-reconstruction
  noise (that requires the dedicated simulate_glv_with_zeros.py workflow)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: SciPy
  role: RK45 Runge-Kutta 5(4) integrator used to forward-simulate gLV ground-truth
    trajectories
- name: NumPy
  role: Finite differences and array computations for trajectory and noise simulation
- name: MGLM
  role: Estimates the Dirichlet-Multinomial dispersion alpha = 251.2809 on the C.
    diff dataset
- name: LUMINATE
  role: 'Downstream consumer: denoised relative abundances estimated from the simulated
    counts via ''python main.py estimate'''
  repo: https://github.com/tyjo/luminate
- name: TGP-CODA
  role: Baseline method run per-sample with default parameters (NUTS sampler in PyStan
    2.17.1.0) on simulated counts
  repo: https://github.com/tare/GPMicrobiome
- name: MALLARD
  role: Baseline method run in RStan 2.19.2 with the canonical (non-phylogenetic)
    ilr basis on simulated counts
  repo: https://github.com/LAD-LAB/MALLARD-Paper-Code
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:5898f9b5c47265e2c099257e9bede910f207055a8871e7b36c8a94b6ef5937c0
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
yes_rate: 0.5625
n_claims: 16
---

# longitudinal-count-noise-simulation

## Summary

Simulates noisy 16S sequencing counts on top of ground-truth longitudinal relative-abundance trajectories (e.g., gLV forward simulations integrated with RK45) using the Silverman et al. framework, with log-normal Poisson depth and an alternative Dirichlet-Multinomial noise model, to benchmark denoising methods like LUMINATE against TGP-CODA and MALLARD.

## When to use

Use when you need synthetic longitudinal count benchmarks with known ground-truth trajectories to evaluate compositional denoising or trajectory-reconstruction methods such as LUMINATE, TGP-CODA, or MALLARD.

## When NOT to use

- Your goal is denoising real observed count data rather than generating synthetic benchmarks — use LUMINATE's 'estimate' command directly instead
- Your data lacks longitudinal structure or known per-mouse initial conditions, so gLV forward simulation cannot be grounded
- You need biological-zero discrimination benchmarks rather than plain trajectory-reconstruction noise (that requires the dedicated simulate_glv_with_zeros.py workflow)

## Inputs

- Longitudinal 16S count/concentration datasets (e.g., C. diff and Diet datasets from bitbucket.org/MDSINE/mdsine)
- Fitted gLV parameters and per-mouse initial conditions
- Noise-model parameters: SNR control matrices Q and R, log-normal depth parameters (log 10000, 0.5), Dirichlet-Multinomial dispersion alpha = 251.2809 (from MGLM)

## Outputs

- Simulated ground-truth relative-abundance trajectories (taxa x time) per mouse/replicate
- Simulated sequencing count tables under log-normal Poisson and Dirichlet-Multinomial noise at varying depths (N = 5000, 10000, 20000)
- Noise-model metadata (SNR, dispersion, depth) for downstream model comparison

## How to apply

Fit gLV parameters per longitudinal dataset (e.g., the C. diff dataset of 5 gnotobiotic mice with 16 species and the Diet dataset of 7 germ-free mice with 17 Clostridia strains from bitbucket.org/MDSINE/mdsine) by discretizing the ODE and performing least squares with elastic net regularization. Forward-simulate ground-truth trajectories from each mouse's initial conditions using the Runge-Kutta 5(4) RK45 integrator from SciPy (5 C. diff trajectories, ~26 time points over 56 days; 7 Diet trajectories, ~47 time points over 65 days). Simulate counts following Silverman et al.: compute the SNR as Tr(Q)/Tr(R) under the additive log-ratio transformation alr(x(t)) = log(x(t)/x_D(t)), set r_i = q_i/SNR, draw z_t ~ N(alr(x_t), R), draw log M_t ~ N(log 10000, 0.5) for log-normal Poisson sequencing depth, and draw y_t ~ Multinomial(N_t, softmax(z_t)). Optionally repeat under Dirichlet-Multinomial noise with dispersion alpha = 251.2809 estimated via the MGLM package on the C. diff dataset (post-convergence observations only), varying sequencing depth N = 5000, 10000, 20000.

## Related tools

- **SciPy** (RK45 Runge-Kutta 5(4) integrator used to forward-simulate gLV ground-truth trajectories)
- **NumPy** (Finite differences and array computations for trajectory and noise simulation)
- **MGLM** (Estimates the Dirichlet-Multinomial dispersion alpha = 251.2809 on the C. diff dataset)
- **LUMINATE** (Downstream consumer: denoised relative abundances estimated from the simulated counts via 'python main.py estimate') — https://github.com/tyjo/luminate
- **TGP-CODA** (Baseline method run per-sample with default parameters (NUTS sampler in PyStan 2.17.1.0) on simulated counts) — https://github.com/tare/GPMicrobiome
- **MALLARD** (Baseline method run in RStan 2.19.2 with the canonical (non-phylogenetic) ilr basis on simulated counts) — https://github.com/LAD-LAB/MALLARD-Paper-Code

## Examples

```
from scipy.integrate import solve_ivp; traj = solve_ivp(glv_rhs, (0, 56), x0_mouse, method='RK45')
```

## Evaluation signals

- Ground-truth trajectories: 5 C. diff trajectories (16 taxa, ~26 time points over 56 days) and 7 Diet trajectories (17 taxa, ~47 time points over 65 days) must be produced
- Simulated count depth follows a log-normal distribution with mean log 10000 and sd 0.5; alternative Dirichlet-Multinomial runs use alpha = 251.2809
- Reconstruction quality: per-taxon mean r2_d = (1/K) * sum_k Corr(p_d(k), p_hat_d(k))^2 between ground truth and estimated trajectories
- Runtime comparison: LUMINATE should be orders of magnitude faster than TGP-CODA and MALLARD at similar or better accuracy

## Limitations

- The log-normal Poisson and Dirichlet-Multinomial noise models do not cover all sequencing artifacts; SNR under the ALR transformation depends on taxon identity and compositional effects
- Filtering out ultra-sparse taxa (>90% missing entries) is recommended as pre-processing because there is little information to reconstruct their trajectory over time
- Both TGP-CODA and MALLARD have difficulty scaling with sample size and taxa, so benchmark runtimes are dominated by those baselines rather than the simulation itself

## Evidence

- [methods] Forward-simulate trajectories with RK45: [evidence span withheld — anchor status: not_found]
- [methods] Simulate sequencing counts under varying noise levels: "We simulated sequencing counts on top of each ground truth trajectory under varying levels of sequencing noise, following the framework of Silverman et al. (2018)"
- [methods] Compute signal-to-noise ratio under additive log-ratio transformation: [evidence span withheld — anchor status: not_found]
- [methods] Alternative Dirichlet-Multinomial noise simulation: [evidence span withheld — anchor status: not_found]
- [methods] MGLM dispersion estimate: [evidence span withheld — anchor status: not_found]
- [readme] Filtering ultra-sparse taxa: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
