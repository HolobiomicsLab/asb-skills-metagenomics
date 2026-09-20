---
name: u-curve-area-under-mixing-curve-calculation
description: Use when multiple binary grouping factors (e.g. location vs gender) are
  available and you need a single quantitative value per factor to rank which factor
  better explains microbiome community differences.
when_to_use_negative:
- You need inferential statistics or FDR-controlled p-values — the paper explicitly
  states AUMC/AOI are descriptive measures and the method does not control the false
  discovery rate
- The grouping factor is not binary — ProgPermute's bin_* functions operate on a binary
  outcome ('variable' denotes the name of binary outcome)
- You only need to identify individual robust features rather than compare overall
  association strength across factors
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_2269
tools:
- name: ProgPermute
  role: R package providing bin_permute_all to run progressive permutation and plot_bin_permute_sigcurve
    to draw the scaled U-curve used for AUMC integration
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Runtime environment in which ProgPermute functions are executed (installed
    via devtools install_github)
- name: R-shiny
  role: Interactive web app (https://biostatistics.mdanderson.org/shinyapps/ProgPerm)
    exposing the U-curve visualization for users who do not run R scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:a8236fe3dcb3558f0f4de49e2b5ce0cfae15ca12e4ec3afda5823390cde73346
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# u-curve-area-under-mixing-curve-calculation

## Summary

Computes the AUMC (area under the mixing curve) from the U-curve of significant features versus proportion of label mixing produced by ProgPermute's progressive permutation, giving a descriptive 0–1 metric for ranking grouping factors by their overall association with the microbiome.

## When to use

Use when multiple binary grouping factors (e.g. location vs gender) are available and you need a single quantitative value per factor to rank which factor better explains microbiome community differences.

## When NOT to use

- You need inferential statistics or FDR-controlled p-values — the paper explicitly states AUMC/AOI are descriptive measures and the method does not control the false discovery rate
- The grouping factor is not binary — ProgPermute's bin_* functions operate on a binary outcome ('variable' denotes the name of binary outcome)
- You only need to identify individual robust features rather than compare overall association strength across factors

## Inputs

- Microbiome feature table with a binary grouping factor (e.g. packaged testdata1 with 'variable' column)
- Binary grouping factor(s) to compare (e.g. location, gender; Dry/Wet vs LDEW/LWED regroupings of SmitsData)

## Outputs

- U-curve (proportion of significant features nsig(k)/p vs proportion of mixing k/K) per grouping factor
- AUMC value per grouping factor (range 0–1)
- Ranked table aumc_ranking.csv with columns: grouping factor, AUMC value, rank

## How to apply

Run progressive permutation for each binary grouping factor via ProgPermute's bin_permute_all (e.g. variable='variable', testdata=testdata1, alpha=0.05, Wilcoxon rank sum test default) to obtain nsig(k), the number of significant features at each permutation scenario k. Scale each nsig(k) by the total feature count p to get the proportion of significant features, and plot this against the proportion of mixing k/K to form the U-curve, whose minimum lies at the full permutation scenario K_f = ceil((n1*n2-1)/(n1+n2+2)). Numerically integrate the curve produced by plot_bin_permute_sigcurve from the observed data (k=0) to the full permutation scenario (k=K_f) to obtain the AUMC, which must lie between 0 and 1. Repeat per grouping factor (e.g. location and gender in the DeFilippo data; Dry/Wet versus LDEW/LWED regroupings of SmitsData), then rank factors in decreasing AUMC order since a bigger AUMC indicates a stronger overall association — the paper's benchmark is location AUMC 0.253 > gender AUMC 0.035. Write the ranked table to aumc_ranking.csv with grouping factor, AUMC, and rank columns.

## Related tools

- **ProgPermute** (R package providing bin_permute_all to run progressive permutation and plot_bin_permute_sigcurve to draw the scaled U-curve used for AUMC integration) — https://github.com/LyonsZhang/ProgPermute
- **R** (Runtime environment in which ProgPermute functions are executed (installed via devtools install_github))
- **R-shiny** (Interactive web app (https://biostatistics.mdanderson.org/shinyapps/ProgPerm) exposing the U-curve visualization for users who do not run R scripts)

## Examples

```
library(ProgPermute); data(testdata1); results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); sigsum<-plot_bin_permute_sigcurve(alloutputs=results,samsize=267,lgndcol=2)
```

## Evaluation signals

- AUMC values must fall between 0 and 1 for every grouping factor
- AUMC should be near 0 (near-flat U-curve) when no true group difference exists (e.g. null-permuted labels)
- Ranking must reproduce the paper's reported ordering, e.g. location AUMC 0.253 > gender AUMC 0.035 in the DeFilippo data application
- The U-curve minimum must occur at the full permutation scenario K_f = ceil((n1*n2-1)/(n1+n2+2))
- aumc_ranking.csv must contain one row per grouping factor with non-empty AUMC and rank columns, sorted by decreasing AUMC

## Limitations

- AUMC is descriptive only — the paper states the measures 'such as AOI or AUMC, are meant to be descriptive' and the paper 'does not come with a formal inference method'
- The method does not control the false discovery rate or calculate adjusted p-values
- top_pm must be less than the number of microbial features, per the README note on bin_permute_all

## Evidence

- [abstract] U-curve definition: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength of the overall association"
- [discussion] AUMC descriptiveness: "The measures we propose, such as AOI or AUMC, are meant to be descriptive"
- [abstract] Wilcoxon default: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [readme] sigcurve invocation: "sigsum<-plot_bin_permute_sigcurve(alloutputs=results,samsize=267,lgndcol=2,psigtitle=NULL,savepsigfile="bin_sigcurve.eps",psigpicdim=c(10,7))"
- [readme] top_pm constraint: ">run progressive permutation. "variable" denotes the name of binary outcome. "top_pm" must be less than the number of microbial features"
