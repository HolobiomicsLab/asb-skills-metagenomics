---
name: clr-compositional-taxa-transform
description: Use when raw genus/OTU count data (e.g. biom.Rdata or otu.tab.txt from
  the MiSurv Data folder) must be converted to compositional predictors before survival
  prediction modeling, such as feeding random survival forests in MiSurv Module 4.
when_to_use_negative:
- The data have already been transformed in the Data Processing module (e.g. rarefaction,
  proportion, or arcsine square root applied) — applying CLR on top would double-transform.
- The analysis goal is Module 1 (comparative survival analysis between treatment groups,
  not involving microbiome data) or Module 2 (comparative microbial composition analysis,
  not involving survival data), where CLR predictors for survival modeling are not
  the target.
- Input is alpha- or beta-diversity output (e.g. 'estimate_richness' in phyloseq or
  'pd' from picante) rather than taxon-level count data.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: MiSurv
  role: Provides the Data Processing module whose data transformations option applies
    the centered log-ratio transform, and Module 4 which consumes the CLR taxa for
    prediction modeling
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: Holds the processed OTU counts, sample data, taxonomy, and tree (biom.Rdata)
    and provides genus-level aggregation and rarefaction (rarefy_even_depth) before
    CLR transformation
- name: randomForestSRC
  role: 'Downstream consumer of the CLR-transformed genus taxa: fits the random survival
    forest (10,000 trees) whose OOB error curve and VIMP ranks validate the transformed
    predictors'
- name: shiny
  role: Launches the MiSurv local app in which the CLR transformation is selected
    interactively
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:a5af13517459af2bfbdd4d2eef0e4f34b8d6f399e34da7f996b7dd50d6f37b1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  - randomForestSRC
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# clr-compositional-taxa-transform

## Summary

Transforms genus-level OTU count tables into centered log-ratio (CLR) compositional predictors, as implemented in the MiSurv Data Processing module, so that taxon abundances can be used as covariates in Module 4 prediction modeling (random survival forests) of survival responses such as time-to-T1D-onset in the Zhang et al. NOD mouse data.

## When to use

Use when raw genus/OTU count data (e.g. biom.Rdata or otu.tab.txt from the MiSurv Data folder) must be converted to compositional predictors before survival prediction modeling, such as feeding random survival forests in MiSurv Module 4.

## When NOT to use

- The data have already been transformed in the Data Processing module (e.g. rarefaction, proportion, or arcsine square root applied) — applying CLR on top would double-transform.
- The analysis goal is Module 1 (comparative survival analysis between treatment groups, not involving microbiome data) or Module 2 (comparative microbial composition analysis, not involving survival data), where CLR predictors for survival modeling are not the target.
- Input is alpha- or beta-diversity output (e.g. 'estimate_richness' in phyloseq or 'pd' from picante) rather than taxon-level count data.

## Inputs

- biom.Rdata (processed phyloseq object from the MiSurvGit Data folder)
- otu.tab.txt (OTU count table)
- sam.dat.txt (sample metadata incl. treatment group and survival time/censoring)
- tax.tab.txt (taxonomy table used for genus-level aggregation)
- tree.tre (phylogenetic tree)

## Outputs

- CLR-transformed genus-level abundance matrix (compositional predictors)
- Predictor matrix consumed by MiSurv Module 4 prediction modeling (e.g. randomForestSRC rfsrc VIMP table and OOB error-rate curve)

## How to apply

Load the processed phyloseq object biom.Rdata from the repository Data folder (or the four individual files otu.tab.txt, sam.dat.txt, tax.tab.txt, and tree.tre) and aggregate OTU counts to the genus level using tax.tab.txt. Optionally rarefy first with the 'rarefy_even_depth' function in the phyloseq package, then apply the centered log-ratio transformation via the MiSurv Data Processing module's data transformations option (alongside alternatives like proportion and arcsine square root). If analyzing treatment effects, subset samples to the target subgroup (e.g. the control group) using sam.dat.txt, since microbiome effects on survival may differ between treatment groups. The resulting CLR-transformed genus table serves as the compositional predictor matrix for Module 4 prediction modeling, e.g. a random survival forest fitted with randomForestSRC::rfsrc against time-to-T1D-onset with 10,000 ensembled trees. Rationale: CLR removes library-size-driven compositionality from count data, giving scale-free taxon covariates suitable for tree-ensemble survival models.

## Related tools

- **MiSurv** (Provides the Data Processing module whose data transformations option applies the centered log-ratio transform, and Module 4 which consumes the CLR taxa for prediction modeling) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (Holds the processed OTU counts, sample data, taxonomy, and tree (biom.Rdata) and provides genus-level aggregation and rarefaction (rarefy_even_depth) before CLR transformation)
- **randomForestSRC** (Downstream consumer of the CLR-transformed genus taxa: fits the random survival forest (10,000 trees) whose OOB error curve and VIMP ranks validate the transformed predictors)
- **shiny** (Launches the MiSurv local app in which the CLR transformation is selected interactively)

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")  # Data Processing > Data Transformations > Centered log-ratio
```

## Evaluation signals

- Per-sample centering invariant: after CLR, the mean of the log-transformed taxon values within each sample is 0 (centered), distinguishing CLR from plain log or proportion transforms.
- Dimension check: the transformed matrix has one column per genus recovered by aggregating otu.tab.txt via tax.tab.txt, and one row per sample retained after subgroup subsetting via sam.dat.txt.
- Downstream validation: the random survival forest fitted on the CLR taxa shows out-of-bag error rates that converge/stabilize by 10,000 ensembled trees (cf. Fig. 7B of the MiSurv paper).
- Ranking check: extracting VIMP from the rfsrc object yields a stable top-10 genus importance table consistent with the paper's Fig. 7A.
- Local runtime check: the environment satisfies the README prerequisite of R >4.2.0 with the shiny package installed before launching MiSurv.

## Limitations

- The web server's hardware specification is limited: 'We will improve the specification if it gets more popular and busier.', so large CLR computations may need the local GitHub implementation.
- No version numbers are given for the R packages used (e.g. phyloseq), and no software license is mentioned for MiSurv, so reproducibility of the transformation stack depends on recent R (>4.2.0) per the README.
- Because microbiome effects on survival are assumed to differ between treatment groups, the CLR taxa should be analyzed within subgroups (e.g. control only), not pooled naively across treatment arms.

## Evidence

- [readme] Data Processing transformations option: "(4) data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root)."
- [intro] Module 4 consumes taxa for prediction modeling: "Module 4: Prediction modeling using microbial taxa on survival responses (Fig. 1)"
- [discussion] Subgroup rationale: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [discussion] Rarefaction before transformation: [evidence span withheld — anchor status: not_found]
