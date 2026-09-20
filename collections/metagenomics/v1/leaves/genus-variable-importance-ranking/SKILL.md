---
name: genus-variable-importance-ranking
description: Use when a survival (time-to-event) analysis needs genus-level predictors
  ranked by predictive influence, e.g., identifying which gut microbial genera most
  strongly predict time-to-T1D-onset in a treatment subgroup.
when_to_use_negative:
- The analysis goal is hypothesis testing of microbiome-survival association rather
  than prediction/ranking — use MiSurv Module 3 association testing instead of the
  Module 4 prediction model
- The question concerns comparative survival between treatment groups without microbiome
  predictors — this is MiSurv Module 1, not prediction modeling
- Input data are not yet processed (no feature table, taxonomy, or survival metadata);
  run MiSurv's Data Processing module (quality control, rarefaction, CLR transformation)
  first
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: randomForestSRC
  role: Fits the random survival forest (rfsrc) with 10,000 trees and provides the
    VIMP variable-importance values and out-of-bag error rates for genus-level predictors
- name: phyloseq
  role: Provides the processed phyloseq object (biom.Rdata) holding the OTU table,
    sample metadata, taxonomy, and tree used to build the genus-level CLR predictor
    matrix
  repo: https://github.com/wg99526/MiSurvGit
- name: MiSurv
  role: Module 4 prediction modeling wraps this workflow; the example data (biom.Rdata
    and individual files) are distributed in the MiSurv GitHub Data folder
  repo: https://github.com/wg99526/MiSurvGit
- name: shiny
  role: Runs the MiSurv app locally (runGitHub("MiSurvGit", "wg99526", ref = "main"))
    to execute Module 4 prediction modeling interactively
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:b9244e65a2c843001715b12a9a38d1b1ce5dd7d10863cfebe3d7e88b6ad317eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  - randomForestSRC
  - shiny
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genus-variable-importance-ranking

## Summary

Ranks microbial genera by their influence on survival prediction using variable importance (VIMP) extracted from a random survival forest (randomForestSRC::rfsrc) fitted on CLR-transformed genus-level abundances, as demonstrated in MiSurv's Module 4 on the Zhang et al. T1D non-obese diabetic mouse gut microbiome data.

## When to use

Use when a survival (time-to-event) analysis needs genus-level predictors ranked by predictive influence, e.g., identifying which gut microbial genera most strongly predict time-to-T1D-onset in a treatment subgroup.

## When NOT to use

- The analysis goal is hypothesis testing of microbiome-survival association rather than prediction/ranking — use MiSurv Module 3 association testing instead of the Module 4 prediction model
- The question concerns comparative survival between treatment groups without microbiome predictors — this is MiSurv Module 1, not prediction modeling
- Input data are not yet processed (no feature table, taxonomy, or survival metadata); run MiSurv's Data Processing module (quality control, rarefaction, CLR transformation) first

## Inputs

- Processed phyloseq object (biom.Rdata) or the four files otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre from the repository Data folder
- Sample metadata (sam.dat.txt) including treatment group, survival time, and censoring/event indicators
- Taxonomy table (tax.tab.txt) for aggregating OTUs to genus level

## Outputs

- Ranked genus variable-importance (VIMP) table; top-10 genera by influence on survival prediction (cf. Fig. 7A)
- Out-of-bag error rate curve by number of trees, confirming convergence by 10,000 trees (cf. Fig. 7B)

## How to apply

Load the processed phyloseq object (biom.Rdata) or the four individual files (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) from the repository Data folder, and subset samples to the treatment subgroup of interest (here, the control group) via the sample metadata in sam.dat.txt. Aggregate OTU counts to the genus level using tax.tab.txt and apply a centered log-ratio (CLR) transformation to obtain compositional predictors. Fit a random survival forest with randomForestSRC::rfsrc on the CLR-transformed genus taxa against the survival response (time-to-T1D-onset with censoring), ensembling 10,000 trees so the out-of-bag error rate converges (verified by plotting the error-rate-by-trees curve, cf. Fig. 7B). Extract variable importance (VIMP) from the fitted rfsrc object, rank genera by importance, and write the top-10 genus importance table (cf. Fig. 7A) and the OOB error curve to output files.

## Related tools

- **randomForestSRC** (Fits the random survival forest (rfsrc) with 10,000 trees and provides the VIMP variable-importance values and out-of-bag error rates for genus-level predictors)
- **phyloseq** (Provides the processed phyloseq object (biom.Rdata) holding the OTU table, sample metadata, taxonomy, and tree used to build the genus-level CLR predictor matrix) — https://github.com/wg99526/MiSurvGit
- **MiSurv** (Module 4 prediction modeling wraps this workflow; the example data (biom.Rdata and individual files) are distributed in the MiSurv GitHub Data folder) — https://github.com/wg99526/MiSurvGit
- **shiny** (Runs the MiSurv app locally (runGitHub("MiSurvGit", "wg99526", ref = "main")) to execute Module 4 prediction modeling interactively) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(randomForestSRC); rsf <- rfsrc(Surv(time, event) ~ ., data = clr_genus_ctrl, ntree = 10000); vimp <- rsf$importance[order(rsf$importance, decreasing = TRUE)]
```

## Evaluation signals

- The out-of-bag error rate curve stabilizes/converges by 10,000 ensembled trees (cf. Fig. 7B); if still decreasing, more trees are needed
- The extracted VIMP table contains all CLR-transformed genus taxa and yields a ranked top-10 list matching the reported influential genera (cf. Fig. 7A)
- Sample subset matches only the control subgroup as defined in sam.dat.txt before model fitting
- Predictor matrix is genus-level (aggregated via tax.tab.txt) and CLR-transformed, not raw counts or proportions
- The fitted rfsrc object uses the survival response (time-to-T1D-onset with censoring indicator) from the sample metadata

## Limitations

- Results are specific to the control (non-antibiotic-treated) subgroup; MiSurv added subgroup analysis because microbiome effects on survival may differ between treatment groups, so VIMP rankings may not transfer across subgroups
- Variable importance indicates predictive influence on survival rates, not a causal or hypothesis-tested association
- MiSurv paper does not report version numbers for R packages used (e.g., phyloseq, survival), which may affect exact reproducibility of VIMP values
- 10,000 trees were needed for OOB error convergence; fewer trees may give unstable importance rankings

## Evidence

- [results] Random survival forests (10,000 trees, control subgroup, CLR data) identified the top 10 genera with the most influential predictions on survival rates (Fig. 7A), and the out-of-bag error rates converged well by 10,000 ensembled trees (Fig. 7B).: [evidence span withheld — anchor status: not_found]
- [intro] Module 4: Prediction modeling using microbial taxa on survival responses: "Module 4: Prediction modeling using microbial taxa on survival responses (Fig. 1)"
- [discussion] we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’): "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root): "data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root)"
