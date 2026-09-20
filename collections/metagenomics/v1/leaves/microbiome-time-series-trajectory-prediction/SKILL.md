---
name: microbiome-time-series-trajectory-prediction
description: Use when longitudinal OTU count tables (taxa x samples with sequence
  IDs and sample times) have been denoised by LUMINATE 'estimate', and the goal is
  to infer cLV parameters (A, g, B) and predict future community trajectories under
  external perturbations.
when_to_use_negative:
- The data are a single cross-sectional sample with no longitudinal replicates, so
  no time-series dynamics or one-step-ahead prediction can be learned
- Raw count data have not been through LUMINATE 'estimate' and the goal is only denoised
  relative abundance or biological-zero detection rather than cLV parameter inference
  and trajectory prediction
- The OTU table contains ultra-sparse taxa (>90% missing entries) that have not been
  filtered; the authors note such taxa carry too little temporal information for trajectory
  reconstruction
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: LUMINATE
  role: Provides the train command to fit cLV parameters (A, g, B) by least squares
    with elastic net regularization, and the experimental predict command (--one-step)
    to produce the predicted-trajectory OTU table
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Used to compute RMSE between predicted and observed trajectories when comparing
    estimate-trained vs pseudo-count-trained models
- name: SciPy
  role: Used for numerical evaluation of predicted trajectories (RMSE per dataset);
    RK45 from SciPy is the Runge-Kutta 5(4) integrator used for forward simulation
    in the gLV analysis
- name: RK45
  role: Runge-Kutta 5(4) numerical integration from SciPy used to forward-simulate
    gLV trajectories given initial conditions of each mouse
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:928d815547b3e1f180bdfbc95a13dc4a3ed21c37441dfcc33b958a0da141fb11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - NumPy
  - RK45
  - SciPy
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# microbiome-time-series-trajectory-prediction

## Summary

Train a compositional Lotka-Volterra (cLV) model on LUMINATE's denoised relative abundance estimates and use it to predict longitudinal community trajectories one step at a time, producing more accurate trajectory predictions than models trained on raw pseudo-counts.

## When to use

Use when longitudinal OTU count tables (taxa x samples with sequence IDs and sample times) have been denoised by LUMINATE 'estimate', and the goal is to infer cLV parameters (A, g, B) and predict future community trajectories under external perturbations.

## When NOT to use

- The data are a single cross-sectional sample with no longitudinal replicates, so no time-series dynamics or one-step-ahead prediction can be learned
- Raw count data have not been through LUMINATE 'estimate' and the goal is only denoised relative abundance or biological-zero detection rather than cLV parameter inference and trajectory prediction
- The OTU table contains ultra-sparse taxa (>90% missing entries) that have not been filtered; the authors note such taxa carry too little temporal information for trajectory reconstruction

## Inputs

- Estimated relative abundances OTU table (e.g. cdiff-counts-est.csv; taxa by samples, first row = longitudinal sequence IDs, second row = sample times, first column = taxon names)
- External events CSV (e.g. cdiff-events.csv with columns sequenceID, eventID, startDay, endDay, magnitude optional)
- P.pkl denoised posterior estimates from LUMINATE 'estimate', located via -i/--indir

## Outputs

- Trained cLV parameters A (taxa interaction matrix), g (relative growth rate vector), B (external perturbation effects), all in ALR coordinates
- Bootstrap one-sided p-values A_pval, g_pval, B_pval (minimum possible value 1/b)
- Predicted-trajectory OTU table (cdiff-counts-est-pred.csv), each trajectory predicted one time point in the future

## How to apply

First run LUMINATE 'estimate' to produce cdiff-counts-est.csv and P.pkl, filtering ultra-sparse taxa (>90% missing entries) beforehand. Run 'python main.py train' on the estimated abundances with the external events CSV (-e) and -i/--indir pointing at the directory containing P.pkl; this fits the discretized generalized Lotka-Volterra model by least squares with elastic net regularization (as in Stein et al., 2013), learning the growth rate vector g, interaction matrix A, and perturbation effects B, output as coordinates under the additive log ratio (ALR) transformation. Pass -b 200 to perform bootstrap resampling and obtain one-sided p-values (A_pval, g_pval, B_pval), where each p-value is the probability that a nonzero coefficient keeps the same sign; the minimum possible p-value is 1/b, so smaller p-values appear as zeros. Then run 'python main.py predict' with the --one-step flag to generate cdiff-counts-est-pred.csv, an OTU table of trajectories predicted one time point in the future (omit --one-step to predict from initial conditions). Validate by comparing predicted vs observed/estimated trajectories via mean r-squared per taxon across samples, or by checking that RMSE of the estimated-abundance-trained model is lower than a pseudo-count-trained baseline (--use-pseudo-counts) on both the C. diff and Diet datasets.

## Related tools

- **LUMINATE** (Provides the train command to fit cLV parameters (A, g, B) by least squares with elastic net regularization, and the experimental predict command (--one-step) to produce the predicted-trajectory OTU table) — https://github.com/tyjo/luminate
- **NumPy** (Used to compute RMSE between predicted and observed trajectories when comparing estimate-trained vs pseudo-count-trained models)
- **SciPy** (Used for numerical evaluation of predicted trajectories (RMSE per dataset); RK45 from SciPy is the Runge-Kutta 5(4) integrator used for forward simulation in the gLV analysis)
- **RK45** (Runge-Kutta 5(4) numerical integration from SciPy used to forward-simulate gLV trajectories given initial conditions of each mouse)

## Examples

```
python main.py predict datasets/bucci2016mdsine/cdiff-counts-est.csv -e datasets/bucci2016mdsine/cdiff-events.csv -i datasets/bucci2016mdsine -o datasets/bucci2016mdsine --one-step
```

## Evaluation signals

- A valid OTU table cdiff-counts-est-pred.csv is written in the expected format (first row sequence IDs, second row sample times, first column taxon names), with each trajectory predicted exactly one time point ahead when --one-step is used
- RMSE of trajectories predicted by the model trained on LUMINATE-estimated abundances is lower than the RMSE of the model trained with --use-pseudo-counts on both the C. diff and Diet datasets
- Mean r-squared per taxon across samples between predicted and observed/estimated trajectories is computed as the reported performance metric
- Bootstrap p-values in A_pval, g_pval, B_pval are bounded below by 1/b (e.g. 1/200 = 0.005); p-values below this threshold appear as zeros, so exact zeros are expected rather than an error
- A, g, B are output as ALR-transformed coordinates (rows under the additive log ratio transformation), matching the paper's parameterization

## Limitations

- Both cLV parameter estimation ('train') and trajectory prediction ('predict') are explicitly flagged as experimental in the LUMINATE README
- The minimum achievable bootstrap p-value is 1/b, so p-values below this threshold are reported as zeros and cannot resolve stronger significance
- Ultra-sparse taxa (>90% missing entries) carry little temporal information and must be filtered as a pre-processing step, otherwise trajectory reconstruction degrades
- If P.pkl is missing from the --indir directory, LUMINATE will run 'estimate' by default; predictions without --one-step only propagate from initial conditions rather than tracking observed data step by step

## Evidence

- [readme] README: predict is one-step-ahead: "This produces an OTU table of predicted trajectories. Each trajectory is predicted one time point in the future."
- [readme] README: bootstrap p-values: "Bootstrap one-sided p-values are in `A_pval`, `g_pval`, `B_pval`. These are the estimated probabilities that the nonzero coefficients estimated on the training data have the same sign."
- [methods] methods: elastic net fitting: [evidence span withheld — anchor status: not_found]
- [methods] methods: mean r2 evaluation metric: "To evaluate model performance, we computed the mean r2, per taxon across samples between the ground truth and estimated trajectories"
- [readme] README: sparsity filtering: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
