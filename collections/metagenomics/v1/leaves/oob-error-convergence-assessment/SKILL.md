---
name: oob-error-convergence-assessment
description: Use when fitting a random survival forest on microbiome taxa for survival
  prediction (MiSurv Module 4) and you must verify OOB error stabilization at a chosen
  ntree (10,000) before ranking predictors by VIMP.
when_to_use_negative:
- When the goal is covariate-adjusted association testing or exploratory survival/composition
  comparison (MiSurv Modules 1-3) rather than ensemble prediction-model validation.
- When taxa are raw counts rather than CLR-transformed compositional abundances —
  apply the CLR transformation before fitting, as in the paper's pipeline.
- When the response is not a censored time-to-event (survival) outcome; MiSurv itself
  is designed specifically for survival responses.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: randomForestSRC
  role: Fits the random survival forest (rfsrc) with 10,000 trees on CLR genus predictors
    and supplies per-iteration OOB error rates and VIMP.
- name: phyloseq
  role: Loads/holds the processed microbiome data (biom.Rdata) and provides genus-level
    aggregation and transformations of OTU counts.
- name: MiSurv
  role: Wraps Module 4 prediction modeling (microbial taxa on survival responses)
    in a shiny web app; the same analysis runs locally via the GitHub repository.
  repo: https://github.com/wg99526/MiSurvGit
- name: shiny
  role: Runs the MiSurv application locally (R >4.2.0 required) to execute Module
    4 with the subgroup and CLR options.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:fe0ffc8475e703fdd6226abf7afb3dc30826764a8f6a3a3beb0a0245a529f748
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  - randomForestSRC
  - shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# oob-error-convergence-assessment

## Summary

Fits a random survival forest (randomForestSRC::rfsrc) on CLR-transformed genus-level microbiome predictors against a censored survival response and traces the out-of-bag (OOB) error rate per tree to verify the ensemble has converged by 10,000 trees before trusting variable-importance rankings.

## When to use

Use when fitting a random survival forest on microbiome taxa for survival prediction (MiSurv Module 4) and you must verify OOB error stabilization at a chosen ntree (10,000) before ranking predictors by VIMP.

## When NOT to use

- When the goal is covariate-adjusted association testing or exploratory survival/composition comparison (MiSurv Modules 1-3) rather than ensemble prediction-model validation.
- When taxa are raw counts rather than CLR-transformed compositional abundances — apply the CLR transformation before fitting, as in the paper's pipeline.
- When the response is not a censored time-to-event (survival) outcome; MiSurv itself is designed specifically for survival responses.

## Inputs

- phyloseq object biom.Rdata (or otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) from the MiSurvGit Data folder
- Sample metadata (sam.dat.txt) with treatment group, survival time, and censored/event indicator
- Genus-level taxonomic mapping (tax.tab.txt) for aggregating OTU counts

## Outputs

- OOB error-rate-by-trees curve (error rate at each of the 10,000 tree iterations)
- Fitted randomForestSRC::rfsrc object on CLR-transformed genus taxa (control subgroup)
- Top-10 genus variable-importance (VIMP) table written to output files

## How to apply

Load the processed phyloseq object (biom.Rdata from the MiSurvGit Data folder, or otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) and subset samples to the control treatment group using sam.dat.txt, since MiSurv adds subgroup analysis because microbiome effects on survival can differ between treatment groups. Aggregate OTU counts to genus via tax.tab.txt and apply a centered log-ratio (CLR) transformation to the genus abundance table so compositional predictors are valid. Fit the random survival forest with randomForestSRC::rfsrc on the CLR genus taxa against the time-to-T1D-onset survival response with ntree = 10,000, then extract the OOB error rate at each tree iteration and plot the error-rate-by-trees curve (cf. Fig. 7B). Judge success visually: the OOB curve should plateau/stabilize well before 10,000 trees; only then extract VIMP, rank genera, and export the top-10 importance table (Fig. 7A) together with the error curve to output files.

## Related tools

- **randomForestSRC** (Fits the random survival forest (rfsrc) with 10,000 trees on CLR genus predictors and supplies per-iteration OOB error rates and VIMP.)
- **phyloseq** (Loads/holds the processed microbiome data (biom.Rdata) and provides genus-level aggregation and transformations of OTU counts.)
- **MiSurv** (Wraps Module 4 prediction modeling (microbial taxa on survival responses) in a shiny web app; the same analysis runs locally via the GitHub repository.) — https://github.com/wg99526/MiSurvGit
- **shiny** (Runs the MiSurv application locally (R >4.2.0 required) to execute Module 4 with the subgroup and CLR options.)

## Examples

```
rsf <- randomForestSRC::rfsrc(Surv(time, event) ~ ., data = clr_genus_control, ntree = 10000)
```

## Evaluation signals

- The OOB error-rate-by-trees curve plateaus and stabilizes well before 10,000 ensembled trees, matching the paper's Fig. 7B convergence behavior.
- The fitted rfsrc object reports a final OOB error rate at ntree = 10,000 on the control-subgroup CLR genus table consistent with the converged value in Fig. 7B.
- The extracted top-10 genus VIMP ranking reproduces the genera shown in Fig. 7A for the control subgroup.
- Error curve behaves as expected for an RSF (decreasing after initial iterations with no divergence at larger ntree).
- Output files contain both the per-iteration OOB error vector/curve and the top-10 importance table.

## Limitations

- Convergence was verified for 10,000 trees on the control subgroup with CLR data; other subgroups, transformations (e.g., arcsine square root), or ntree settings may require re-checking the curve.
- The README omits R package versions (e.g., randomForestSRC, phyloseq), so exact reproducibility of the OOB curve may vary across package versions.
- MiSurv notes hardware capacity limits on the web server; a 10,000-tree forest may need the local GitHub implementation for compute-heavy runs.

## Evidence

- [methods] Fit a random survival forest with randomForestSRC::rfsrc on the CLR-transformed genus taxa against the survival (time-to-T1D-onset) response, ensembling 10,000 trees.: [evidence span withheld — anchor status: not_found]
- [results] the out-of-bag error rates converged well by 10,000 ensembled trees (Fig. 7B).: [evidence span withheld — anchor status: not_found]
- [readme] Prediction modeling using microbial taxa at different taxonomic ranks on survival responses.: "Prediction modeling using microbial taxa at different taxonomic ranks on survival responses."
