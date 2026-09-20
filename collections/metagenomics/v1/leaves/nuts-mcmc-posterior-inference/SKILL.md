---
name: nuts-mcmc-posterior-inference
description: Use when a Bayesian compositional time-series model (e.g. TGP-CODA or
  MALLARD) must be fit by MCMC to longitudinal 16S count data to obtain posterior
  estimates of latent relative abundances for accuracy/runtime benchmarking.
when_to_use_negative:
- When the goal is fast inference from large numbers of samples or many taxa — TGP-CODA
  and MALLARD have difficulty scaling with sample size and taxa, and the paper shows
  they are orders of magnitude slower than LUMINATE
- When the input is already denoised relative abundances rather than raw sequencing
  counts
- When a non-Bayesian point-estimate method (e.g. LUMINATE's variational inference)
  suffices for the analysis
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_2269
tools:
- name: PyStan 2.17.1.0
  role: Python interface to Stan providing the NUTS sampler used to fit TGP-CODA per
    sample with default parameters
  repo: https://github.com/tare/GPMicrobiome
- name: RStan 2.19.2
  role: R interface to Stan used to run the MALLARD model with the canonical ilr basis
    and default initialization
  repo: https://github.com/LAD-LAB/MALLARD-Paper-Code
- name: TGP-CODA
  role: Temporal probabilistic model of bacterial compositions fit via NUTS in PyStan
    as a baseline
  repo: https://github.com/tare/GPMicrobiome
- name: MALLARD
  role: Bayesian longitudinal count model run in RStan as a baseline for trajectory
    reconstruction
  repo: https://github.com/LAD-LAB/MALLARD-Paper-Code
- name: LUMINATE
  role: Fast variational-inference alternative whose estimates and runtimes are compared
    against the NUTS-based baselines
  repo: https://github.com/tyjo/luminate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:f5d7878a9e6bab975cc8a15e881776b31172479f7cafa10505dede48d8069c63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tare/GPMicrobiome
  tools:
  - LUMINATE
  - MALLARD
  - MGLM
  - NumPy
  - PyStan
  - PyStan 2.17.1.0
  - RK45
  - RStan
  - RStan 2.19.2
  - SciPy
  - TGP-CODA
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# nuts-mcmc-posterior-inference

## Summary

Run No-U-Turn Sampler (NUTS) posterior inference for latent relative abundances in Bayesian compositional state-space models (TGP-CODA via PyStan 2.17.1.0, MALLARD via RStan 2.19.2) on longitudinal 16S count data, taking the posterior mean as the denoised trajectory estimate.

## When to use

Use when a Bayesian compositional time-series model (e.g. TGP-CODA or MALLARD) must be fit by MCMC to longitudinal 16S count data to obtain posterior estimates of latent relative abundances for accuracy/runtime benchmarking.

## When NOT to use

- When the goal is fast inference from large numbers of samples or many taxa — TGP-CODA and MALLARD have difficulty scaling with sample size and taxa, and the paper shows they are orders of magnitude slower than LUMINATE
- When the input is already denoised relative abundances rather than raw sequencing counts
- When a non-Bayesian point-estimate method (e.g. LUMINATE's variational inference) suffices for the analysis

## Inputs

- Longitudinal 16S OTU/count table (taxa x samples) with sequence IDs, sample times, and taxon names (e.g. datasets/bucci2016mdsine/cdiff-counts.csv)
- Stan model files for TGP-CODA (PyStan 2.17.1.0) and MALLARD (RStan 2.19.2, canonical non-phylogenetic ilr basis)

## Outputs

- Posterior samples of latent relative abundances (posterior mean taken as point estimate)
- Denoised relative abundance trajectories per taxon per sample
- Wall-clock runtimes per model run for benchmarking against LUMINATE

## How to apply

Fit the model per sample: run TGP-CODA (from github.com/tare/GPMicrobiome) individually on each sample with default parameters using the NUTS sampler in PyStan 2.17.1.0, and run MALLARD (from github.com/LAD-LAB/MALLARD-Paper-Code) in RStan 2.19.2 using the canonical (non-phylogenetic) ilr basis with RStan default initialization. For LUMINATE-style models, estimate latent relative abundances by taking the mean of the posterior samples of variables QG computed with NUTS in PyStan. Record wall-clock runtimes alongside posterior estimates, since scaling with sample size and taxa is a key limitation of these Stan-based baselines. Evaluate quality by computing the mean r-squared per taxon across samples, r2_d = (1/K) * sum_k Corr(p_d(k), p_hat_d(k))^2, against ground-truth trajectories from synthetic data (e.g. RK45-forward-simulated gLV trajectories from the C. diff and Diet datasets).

## Related tools

- **PyStan 2.17.1.0** (Python interface to Stan providing the NUTS sampler used to fit TGP-CODA per sample with default parameters) — https://github.com/tare/GPMicrobiome
- **RStan 2.19.2** (R interface to Stan used to run the MALLARD model with the canonical ilr basis and default initialization) — https://github.com/LAD-LAB/MALLARD-Paper-Code
- **TGP-CODA** (Temporal probabilistic model of bacterial compositions fit via NUTS in PyStan as a baseline) — https://github.com/tare/GPMicrobiome
- **MALLARD** (Bayesian longitudinal count model run in RStan as a baseline for trajectory reconstruction) — https://github.com/LAD-LAB/MALLARD-Paper-Code
- **LUMINATE** (Fast variational-inference alternative whose estimates and runtimes are compared against the NUTS-based baselines) — https://github.com/tyjo/luminate

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- Mean per-taxon r-squared r2_d = (1/K) * sum_k Corr(p_d(k), p_hat_d(k))^2 computed against ground-truth trajectories across samples should be comparable to or better than competing methods
- Wall-clock runtime should be recorded and compared against LUMINATE; NUTS-based baselines are expected to be orders of magnitude slower
- Posterior estimates are the mean of posterior samples of variables QG; verify output is a valid relative-abundance trajectory (compositional, per time point)
- MALLARD must be run with the canonical (non-phylogenetic) ilr basis and RStan default initialization for reproducibility of the benchmark
- Each TGP-CODA fit must be run on one sample individually with default parameters

## Limitations

- TGP-CODA and MALLARD have difficulty scaling with sample size and taxa; they are orders of magnitude slower than LUMINATE
- Posterior inference from time-series data is computationally challenging, motivating variational approaches
- For ultra-sparse taxa (>90% missing entries) there is little information about the trajectory over time; such taxa should be filtered as a pre-processing step

## Evidence

- [methods] NUTS in PyStan: [evidence span withheld — anchor status: not_found]
- [methods] TGP-CODA run per sample: [evidence span withheld — anchor status: not_found]
- [methods] MALLARD in RStan: [evidence span withheld — anchor status: not_found]
- [intro] scaling limitation: "However, efficient inference from time-series data is a challenging problem, and both methods have difficulty scaling with sample size and taxa."
- [methods] evaluation metric: "To evaluate model performance, we computed the mean r2, per taxon across samples between the ground truth and estimated trajectories"
- [readme] sparsity filter: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
