---
name: random-survival-forest-modeling
description: Use when the goal is to predict a survival (time-to-event) response from
  compositional microbiome predictors — e.g., ranking microbial genera by variable
  importance and verifying out-of-bag error convergence by 10,000 ensembled trees.
when_to_use_negative:
- The goal is association testing between microbial composition and survival (MiSurv
  Module 3, e.g. MiRKAT-style testing) rather than prediction modeling — use the association-testing
  skill instead
- The goal is exploratory comparison of survival curves between treatment groups (MiSurv
  Module 1), which does not involve microbiome data
- The response is continuous or binary rather than a survival (time-to-event) response
  — MiSurv is specifically designed for survival responses while MiCloud handles continuous/binary
  outcomes
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: randomForestSRC
  role: Fits the random survival forest (rfsrc) with 10,000 trees on CLR-transformed
    genus taxa; provides VIMP and per-iteration out-of-bag error rates
- name: MiSurv
  role: Web cloud platform whose Module 4 performs prediction modeling using microbial
    taxa on survival responses, with subgroup analysis and CLR data transformation
    options
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: Handles the phyloseq-format input (biom.Rdata) and supports genus-level aggregation
    and CLR transformation of the abundance table before modeling
- name: shiny
  role: R framework required to launch the MiSurv app locally via runGitHub("MiSurvGit",
    "wg99526", ref = "main")
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:4f17516c96be92b8bc5e8a85a9345c251776ce1ae6f8cd997e6c41f8c8509e3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  - randomForestSRC
  - shiny
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# random-survival-forest-modeling

## Summary

Prediction modeling of survival (time-to-T1D-onset) from CLR-transformed genus-level microbiome taxa using random survival forests ensembling 10,000 trees, as implemented in MiSurv Module 4 with randomForestSRC::rfsrc.

## When to use

Use when the goal is to predict a survival (time-to-event) response from compositional microbiome predictors — e.g., ranking microbial genera by variable importance and verifying out-of-bag error convergence by 10,000 ensembled trees.

## When NOT to use

- The goal is association testing between microbial composition and survival (MiSurv Module 3, e.g. MiRKAT-style testing) rather than prediction modeling — use the association-testing skill instead
- The goal is exploratory comparison of survival curves between treatment groups (MiSurv Module 1), which does not involve microbiome data
- The response is continuous or binary rather than a survival (time-to-event) response — MiSurv is specifically designed for survival responses while MiCloud handles continuous/binary outcomes

## Inputs

- biom.Rdata (phyloseq object from the MiSurvGit Data folder)
- otu.tab.txt (OTU count table)
- sam.dat.txt (sample metadata incl. survival time, censoring, treatment group)
- tax.tab.txt (taxonomy table used for genus-level aggregation)
- tree.tre (phylogenetic tree)

## Outputs

- Top-10 genus variable-importance (VIMP) table ranked by prediction influence
- Out-of-bag error-rate-by-trees curve showing convergence by 10,000 trees
- Fitted rfsrc object (VIMP and OOB error rate at each tree iteration)

## How to apply

Load the processed phyloseq object biom.Rdata from the MiSurvGit Data folder (or the equivalent otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre files). Subset samples to the control treatment group using sam.dat.txt, since microbiome effects on survival can differ between treatment groups (MiSurv's subgroup-analysis option). Aggregate OTU counts to genus level via tax.tab.txt and apply a centered log-ratio (CLR) transformation to the genus abundance table to obtain compositional predictors. Fit a random survival forest with randomForestSRC::rfsrc on the CLR-transformed genus taxa against the survival response, ensembling 10,000 trees. Extract the out-of-bag error rate at each tree iteration and plot the error-rate-by-trees curve, confirming it converges/stabilizes by 10,000 trees (cf. Fig. 7B of the T1D example). Finally extract variable importance (VIMP), rank genera, and write the top-10 genus importance table and OOB curve to output files (cf. Fig. 7A).

## Related tools

- **randomForestSRC** (Fits the random survival forest (rfsrc) with 10,000 trees on CLR-transformed genus taxa; provides VIMP and per-iteration out-of-bag error rates)
- **MiSurv** (Web cloud platform whose Module 4 performs prediction modeling using microbial taxa on survival responses, with subgroup analysis and CLR data transformation options) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (Handles the phyloseq-format input (biom.Rdata) and supports genus-level aggregation and CLR transformation of the abundance table before modeling)
- **shiny** (R framework required to launch the MiSurv app locally via runGitHub("MiSurvGit", "wg99526", ref = "main"))

## Examples

```
rsf.fit <- randomForestSRC::rfsrc(Surv(time_to_T1D_onset, event) ~ ., data = clr_genus_ctrl, ntree = 10000)
```

## Evaluation signals

- The OOB error-rate-by-trees curve plateaus/stabilizes well before or by 10,000 ensembled trees (cf. Fig. 7B); if still declining, more trees are needed
- The ranked VIMP table reproduces a top-10 genus set consistent with Fig. 7A of the T1D example
- The fitted rfsrc object exposes err.rate at each of the 10,000 tree iterations and a vimp vector over all CLR-transformed genus predictors
- Sample subsetting via sam.dat.txt leaves only the control treatment group (the subgroup in which the top-10 genera were identified)
- Predictors are CLR-transformed genus-level abundances (not raw counts or proportions), matching the compositional-data treatment in the workflow

## Limitations

- OOB convergence is verified visually (error-rate-by-trees curve), not by a formal numeric stopping rule
- Results are subgroup-specific: the analysis was added because effects of microbiome on survival responses are often assumed to differ between treatment groups, so pooled modeling may obscure group-specific predictors
- Underlying data are 16S rRNA V4 OTUs clustered at 97% similarity against Greengenes, so predictor resolution is genus-level compositional counts, not absolute abundances
- No version numbers for R packages (e.g., phyloseq, randomForestSRC) are documented in the article or README, complicating exact reproduction
- MiSurv web server hardware capacity is limited; large ensemble fits may need local implementation via the GitHub repository

## Evidence

- [intro] Module 4: Prediction modeling using microbial taxa on survival responses (Fig. 1): "Module 4: Prediction modeling using microbial taxa on survival responses (Fig. 1)"
- [discussion] we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’): "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [abstract] MiSurv consists of a data processing module and its following four data analytic modules: "MiSurv consists of a data processing module and its following four data analytic modules"
- [discussion] the rarefaction (50) using the "rarefy_even_depth" in the "phyloseq" package: [evidence span withheld — anchor status: not_found]
