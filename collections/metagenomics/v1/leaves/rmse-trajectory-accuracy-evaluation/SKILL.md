---
name: rmse-trajectory-accuracy-evaluation
description: Use when comparing two dynamical-system parameterizations (e.g., trained
  on denoised vs raw/pseudo-count abundances) by measuring RMSE of predicted versus
  observed community trajectories over time.
when_to_use_negative:
- The model output is not a time-series trajectory prediction (e.g., static cross-sectional
  abundance estimates) — RMSE over trajectories is meaningless without a temporal
  dimension
- Evaluating zero-detection performance (biological vs technical zeros), where posterior
  probabilities from LUMINATE's `estimate` step are the relevant output instead
- Trajectories contain ultra-sparse taxa (>90% missing entries) that were not filtered,
  since these have little information and can distort RMSE comparisons
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LUMINATE
  role: Provides the estimated (denoised) relative abundances used to train cLV and
    the `predict --one-step` command producing the predicted trajectories evaluated
    by RMSE
  repo: https://github.com/tyjo/luminate
- name: NumPy
  role: Numerical computation of RMSE between predicted and observed trajectories
- name: SciPy
  role: Supplies numerical routines used in trajectory evaluation; RK45 from SciPy
    is used for Runge-Kutta 5(4) forward simulation of trajectories
- name: RK45
  role: Runge-Kutta 5(4) numerical integration method (from SciPy) used to forward-simulate
    trajectories given initial conditions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cels.2020.05.006
    title: Efficient and Accurate Inference of Mixed Microbial Population Trajectories
      from Longitudinal Count Data
  source_content_hash: sha256:c1b141765357964b7e6ab861c7c48dadae89305c4b4f5fd35efaa82b30741f4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyjo/luminate
  tools:
  - LUMINATE
  - NumPy
  - RK45
  - SciPy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# rmse-trajectory-accuracy-evaluation

## Summary

Evaluates dynamical-system (cLV/gLV) trajectory prediction accuracy by computing root-mean-square-error (RMSE) between one-step-ahead predicted and observed taxa trajectories, here to show that cLV trained on LUMINATE-estimated relative abundances outperforms pseudo-count-trained baselines on the C. diff and Diet datasets.

## When to use

Use when comparing two dynamical-system parameterizations (e.g., trained on denoised vs raw/pseudo-count abundances) by measuring RMSE of predicted versus observed community trajectories over time.

## When NOT to use

- The model output is not a time-series trajectory prediction (e.g., static cross-sectional abundance estimates) — RMSE over trajectories is meaningless without a temporal dimension
- Evaluating zero-detection performance (biological vs technical zeros), where posterior probabilities from LUMINATE's `estimate` step are the relevant output instead
- Trajectories contain ultra-sparse taxa (>90% missing entries) that were not filtered, since these have little information and can distort RMSE comparisons

## Inputs

- One-step-ahead predicted OTU table from `python main.py predict --one-step` (e.g., cdiff-counts-est-pred.csv; taxa by samples with sequence IDs in row 1, sample times in row 2, taxon names in column 1)
- Observed OTU trajectory table (e.g., datasets/bucci2016mdsine/cdiff-counts.csv) or LUMINATE-estimated abundances (cdiff-counts-est.csv) as the reference trajectories
- Fitted cLV parameters A (interaction matrix), g (growth vector), B (perturbation matrix) for each trained model (estimate-trained vs pseudo-count-trained)

## Outputs

- Per-dataset RMSE values comparing predicted vs observed trajectories for the estimate-trained and pseudo-count-trained cLV models
- Valid predicted OTU table (cdiff-counts-est-pred.csv) confirming the predict step ran correctly

## How to apply

After training cLV models (one on LUMINATE-estimated abundances from cdiff-counts-est.csv, one via the --use-pseudo-counts flag on raw counts), run `python main.py predict --one-step` to produce a one-step-ahead predicted OTU table (e.g., cdiff-counts-est-pred.csv). Then compute per-dataset RMSE between the predicted and observed trajectories using NumPy/SciPy, comparing the estimate-trained model against the pseudo-count-trained model on both the C. diff (Bucci et al., 2016, 16 species) and Diet (17 Clostridia strains) datasets. The rationale is that a model fitted on denoised relative abundances should better recapitulate true community dynamics, and RMSE quantifies trajectory-level accuracy. For forward simulation from initial conditions, trajectories are integrated with the Runge-Kutta 5(4) method (RK45 in SciPy). Validation requires the estimate-trained RMSE to be lower than the pseudo-count-trained RMSE on both datasets, with a valid OTU table written as output.

## Related tools

- **LUMINATE** (Provides the estimated (denoised) relative abundances used to train cLV and the `predict --one-step` command producing the predicted trajectories evaluated by RMSE) — https://github.com/tyjo/luminate
- **NumPy** (Numerical computation of RMSE between predicted and observed trajectories)
- **SciPy** (Supplies numerical routines used in trajectory evaluation; RK45 from SciPy is used for Runge-Kutta 5(4) forward simulation of trajectories)
- **RK45** (Runge-Kutta 5(4) numerical integration method (from SciPy) used to forward-simulate trajectories given initial conditions)

## Examples

```
python main.py predict datasets/bucci2016mdsine/cdiff-counts-est.csv -e datasets/bucci2016mdsine/cdiff-events.csv -i datasets/bucci2016mdsine -o datasets/bucci2016mdsine --one-step
```

## Evaluation signals

- RMSE of trajectories predicted from the model trained on LUMINATE-estimated abundances must be lower than the RMSE from the pseudo-count-trained model on both the C. diff and Diet datasets
- A valid predicted OTU table (cdiff-counts-est-pred.csv) is written with the expected schema (sequence IDs row, sample times row, taxon names column)
- The predicted table covers the same taxa and longitudinal samples as the input estimated-abundance table, one time point ahead per sample
- If forward-simulating from initial conditions, integrated trajectories are reproducible with RK45 from SciPy using the fitted A, g, B parameters

## Limitations

- Parameter estimation of cLV with LUMINATE is explicitly considered experimental by the authors, and `predict` is flagged as experimental in the README
- Ultra-sparse taxa (>90% missing entries) carry little trajectory information and are recommended to be filtered before training/evaluation
- The demonstration is on the two Bucci et al. (2016) mouse datasets (C. diff: 5 gnotobiotic mice, 16 species; Diet: 7 germ-free mice, 17 Clostridia strains); performance on other datasets is not established
- One-step-ahead prediction differs from full forward simulation from initial conditions; choose `--one-step` vs omitting the flag depending on the evaluation goal

## Evidence

- [intro] Finally, we demonstrate the utility of LUMINATE by using estimated relative abundances to infer the parameters of a dynamical system, leading to more accurate predictions of community trajectories.: "we demonstrate the utility of LUMINATE by using estimated relative abundances to infer the parameters of a dynamical system, leading to more accurate predictions of com- munity trajectories"
- [methods] gLV fitting by discretized least squares with elastic net regularization: [evidence span withheld — anchor status: not_found]
- [methods] RK45 forward simulation of trajectories: [evidence span withheld — anchor status: not_found]
- [readme] README: predict --one-step produces predicted trajectories one time point ahead: "This produces an OTU table of predicted trajectories. Each trajectory is predicted one time point in the future."
- [readme] README: --use-pseudo-counts flag trains on pseudo-counts instead of denoised estimates (baseline for RMSE comparison): "To train model parameters using pseudo-counts instead of denoised estimates, pass the --use-pseudo-counts flag instead."
- [readme] README: filtering ultra-sparse taxa with >90% missing entries before trajectory analysis: "For ultra-sparse taxa (>90% missing entries), there is little information its trajectory over time. We recommend filtering out such taxa as a pre-processing step."
