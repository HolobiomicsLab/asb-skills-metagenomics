---
name: per-taxon-accuracy-benchmarking
description: Use when benchmarking trajectory-reconstruction accuracy of denoised
  relative abundances per taxon across samples, e.g. comparing LUMINATE against TGP-CODA/MALLARD
  on simulated longitudinal count data.
when_to_use_negative:
- You need to compare methods on biological zero detection or cLV parameter fitting
  rather than relative abundance trajectory accuracy — this skill only scores per-taxon
  reconstruction accuracy and runtime
- No ground-truth trajectories exist (e.g. purely real observational data) — the squared-correlation-per-taxon
  metric requires known true relative abundances
- Datasets contain ultra-sparse taxa (>90% missing entries), which the paper recommends
  filtering out as a pre-processing step before benchmarking
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: LUMINATE
  role: Method under test; run 'python main.py estimate' on each simulated dataset
    to obtain denoised relative abundances
  repo: https://github.com/tyjo/luminate
- name: TGP-CODA
  role: Baseline method run per sample with default parameters using NUTS in PyStan
    2.17.1.0
  repo: https://github.com/tare/GPMicrobiome
- name: MALLARD
  role: Baseline method run in RStan 2.19.2 with the canonical (non-phylogenetic)
    ilr basis and default initialization
  repo: https://github.com/LAD-LAB/MALLARD-Paper-Code
- name: PyStan
  role: Stan Python interface used to run TGP-CODA's No-U-Turn Sampler
- name: RStan
  role: Stan R interface used to run the MALLARD model
- name: MGLM
  role: Used to estimate Dirichlet-Multinomial dispersion (alpha = 251.2809) on the
    C. diff dataset for the alternative noise model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:076102310b63c7438260de2ed5f6206726a6599a310c43db00ad3c6a6ee87fb1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# per-taxon-accuracy-benchmarking

## Summary

Evaluates and compares methods that reconstruct latent relative abundances from longitudinal 16S count data by computing a mean per-taxon r-squared between ground-truth and estimated trajectories, alongside wall-clock runtime comparisons (e.g., LUMINATE vs TGP-CODA vs MALLARD).

## When to use

Use when benchmarking trajectory-reconstruction accuracy of denoised relative abundances per taxon across samples, e.g. comparing LUMINATE against TGP-CODA/MALLARD on simulated longitudinal count data.

## When NOT to use

- You need to compare methods on biological zero detection or cLV parameter fitting rather than relative abundance trajectory accuracy — this skill only scores per-taxon reconstruction accuracy and runtime
- No ground-truth trajectories exist (e.g. purely real observational data) — the squared-correlation-per-taxon metric requires known true relative abundances
- Datasets contain ultra-sparse taxa (>90% missing entries), which the paper recommends filtering out as a pre-processing step before benchmarking

## Inputs

- Ground-truth relative abundance trajectories from forward-simulated gLV dynamics (e.g. 5 C. diff trajectories of 16 taxa, 7 Diet trajectories of 17 taxa)
- Simulated sequencing count tables under log-normal Poisson and Dirichlet-Multinomial (alpha = 251.2809) noise at depths N = 5000, 10000, 20000
- Estimated relative abundances produced by each compared model (LUMINATE, TGP-CODA, MALLARD)
- Wall-clock runtime measurements per model per dataset

## Outputs

- Per-sample, per-taxon mean r-squared values (r2_d = (1/K) * sum_k Corr(p_d(k), p_hat_d(k))^2) for each model
- Wall-clock runtimes for LUMINATE, TGP-CODA, and MALLARD
- Compiled comparison table (model_comparison.csv) covering all three models, both noise models, and all sequencing depths

## How to apply

Run each candidate model (LUMINATE via 'estimate', TGP-CODA with the NUTS sampler in PyStan 2.17.1.0 on each sample individually, and MALLARD in RStan 2.19.2 using the canonical non-phylogenetic ilr basis with default initialization) on the same simulated datasets. For each sample d, compute the per-taxon accuracy as r2_d = (1/K) * sum_k Corr(p_d(k), p_hat_d(k))^2, i.e. the squared Pearson correlation between ground-truth and estimated relative abundances averaged across the K taxa. Evaluate over both simulated noise models (log-normal Poisson sequencing depth and Dirichlet-Multinomial with dispersion alpha = 251.2809 estimated by MGLM on the C. diff dataset) and sequencing depths N = 5000, 10000, 20000. Record wall-clock runtimes per model so accuracy can be weighed against computational cost, then compile per-taxon accuracy and runtime for all models, noise models, and depths into a single comparison table (model_comparison.csv).

## Related tools

- **LUMINATE** (Method under test; run 'python main.py estimate' on each simulated dataset to obtain denoised relative abundances) — https://github.com/tyjo/luminate
- **TGP-CODA** (Baseline method run per sample with default parameters using NUTS in PyStan 2.17.1.0) — https://github.com/tare/GPMicrobiome
- **MALLARD** (Baseline method run in RStan 2.19.2 with the canonical (non-phylogenetic) ilr basis and default initialization) — https://github.com/LAD-LAB/MALLARD-Paper-Code
- **PyStan** (Stan Python interface used to run TGP-CODA's No-U-Turn Sampler)
- **RStan** (Stan R interface used to run the MALLARD model)
- **MGLM** (Used to estimate Dirichlet-Multinomial dispersion (alpha = 251.2809) on the C. diff dataset for the alternative noise model)

## Examples

```
python main.py estimate datasets/bucci2016mdsine/cdiff-counts.csv -e datasets/bucci2016mdsine/cdiff-events.csv -o datasets/bucci2016mdsine
```

## Evaluation signals

- Every r2_d value lies in [0, 1] because it is an average of squared per-taxon Pearson correlations between ground truth and estimates
- model_comparison.csv contains entries for all three models (LUMINATE, TGP-CODA, MALLARD), both noise models (log-normal Poisson and Dirichlet-Multinomial with alpha = 251.2809), and all three depths (N = 5000, 10000, 20000)
- Expected result pattern: LUMINATE is orders of magnitude faster than TGP-CODA and MALLARD while achieving better or similar accuracy
- TGP-CODA is run individually per sample with default parameters and MALLARD with the canonical ilr basis — deviations from these settings invalidate cross-model comparability
- Runtimes are wall-clock times recorded under the same hardware/conditions for all models

## Limitations

- TGP-CODA and MALLARD have difficulty scaling with sample size and taxa, so runtime comparisons may be infeasible for large datasets
- Ultra-sparse taxa (>90% missing entries) carry little trajectory information and are recommended for pre-processing removal
- There is currently no public implementation of MALLARDs in the Silverman et al. framework, limiting extension of the benchmark to that variant

## Evidence

- [methods] mean r2 per taxon across samples: "To evaluate model performance, we computed the mean r2, per taxon across samples between the ground truth and estimated trajectories"
- [abstract] orders of magnitude faster: "We demonstrate that LUMINATE is orders of magnitude faster than current approaches, with better or similar accuracy."
- [intro] scaling difficulty of baselines: "However, efficient inference from time-series data is a challenging problem, and both methods have difficulty scaling with sample size and taxa."
- [readme] sparse-taxa filtering caveat: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time."
- [methods] NUTS posterior inference for TGP-CODA: [evidence span withheld — anchor status: not_found]
