---
name: progressive-permutation-label-exchange
description: Use when a microbiome feature table has a binary grouping factor and
  you need to assess the robustness/overall association of differential abundance
  discoveries by progressively permuting group labels and tracking significant-feature
  counts across mixing levels.
when_to_use_negative:
- The task requires formal inference, FDR-controlled or adjusted p-values — the article
  states the method 'does not come with a formal inference method' and does not control
  the false discovery rate
- The goal is comparing grouping factors via ordination/PERMANOVA rather than robustness
  of individual differential features (e.g., PCoA/NMDS or PERMANOVA analysis)
- The grouping factor is not a binary outcome that bin_permute_all can progressively
  permute
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ProgPermute
  role: R package providing bin_permute_all for progressive label exchange and plot_bin_psig
    for U-curve visualization
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Language/environment in which the progressive permutation and plotting functions
    are executed
- name: R-shiny
  role: User-friendly web interface wrapping the progressive permutation workflow
    with visualizations (ShinyApp at https://biostatistics.mdanderson.org/shinyapps/ProgPerm)
  repo: https://github.com/LyonsZhang/ProgPerm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:534b261154a5d65e4e58d1546db9f0a12002ab92d0bcd686ac287191954ca9a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# progressive-permutation-label-exchange

## Summary

Applies the ProgPermute progressive permutation method: it progressively exchanges grouping-factor labels between groups and repeats differential abundance testing (default Wilcoxon rank sum test, alpha=0.05) at each mixing level, producing a characteristic U-curve of significant-feature counts versus proportion of mixing whose shape conveys the strength of the overall microbiome-group association.

## When to use

Use when a microbiome feature table has a binary grouping factor and you need to assess the robustness/overall association of differential abundance discoveries by progressively permuting group labels and tracking significant-feature counts across mixing levels.

## When NOT to use

- The task requires formal inference, FDR-controlled or adjusted p-values — the article states the method 'does not come with a formal inference method' and does not control the false discovery rate
- The goal is comparing grouping factors via ordination/PERMANOVA rather than robustness of individual differential features (e.g., PCoA/NMDS or PERMANOVA analysis)
- The grouping factor is not a binary outcome that bin_permute_all can progressively permute

## Inputs

- Microbiome count/abundance feature table with a binary grouping variable column (e.g., testdata1 loaded via data(testdata1))
- Name of the binary outcome variable ('variable')
- top_pm parameter (number of top features to track; must be less than the number of microbial features)
- zoomn (zoom count), alpha (significance level, 0.05), and plot y-range parameters (e.g., psigyrange=c(0,170))

## Outputs

- results object from bin_permute_all containing per-scenario differential test outputs
- U-curve plot of number of significant features versus proportion of mixing (sigfeatures.eps)
- Per-scenario count of significant taxa nsig(k)

## How to apply

Load the ProgPermute package (install_github('LyonsZhang/ProgPermute')) in R and load the example data with data(testdata1). Run the progressive permutation with bin_permute_all(variable='variable', testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), where 'top_pm' must be less than the number of microbial features and the default Wilcoxon rank sum test computes p-values at alpha=0.05 in each scenario. Summarize per-scenario median p-values into the count of significant taxa nsig(k) at each progressive permutation scenario. Generate the U-curve of number of significant features versus proportion of mixing via plot_bin_psig(alloutputs=results, psigyrange=c(0,170), savepsigfile='sigfeatures.eps'). Judge success by whether the plot shows a decreasing-then-U-shaped trend from the observed data toward the fully mixed scenario: the decreasing trend indicates top hits are true positives, and the U-curve shape conveys the strength of the overall association between microbiome and the grouping factor.

## Related tools

- **ProgPermute** (R package providing bin_permute_all for progressive label exchange and plot_bin_psig for U-curve visualization) — https://github.com/LyonsZhang/ProgPermute
- **R** (Language/environment in which the progressive permutation and plotting functions are executed)
- **R-shiny** (User-friendly web interface wrapping the progressive permutation workflow with visualizations (ShinyApp at https://biostatistics.mdanderson.org/shinyapps/ProgPerm)) — https://github.com/LyonsZhang/ProgPerm

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); sigloc<-plot_bin_psig(alloutputs=results,psigyrange=c(0,170),savepsigfile="sigfeatures.eps")
```

## Evaluation signals

- The saved sigfeatures.eps exists and is a valid EPS plot produced by plot_bin_psig with psigyrange=c(0,170)
- The plot shows a decreasing trend from the observed data toward intermediate mixing levels, then a U-shaped recovery toward the fully permuted scenario, comparable to the deposited UCurve.png reference
- top_pm (267) is confirmed to be less than the total number of microbial features in testdata1, as required by bin_permute_all
- Per-scenario significant counts nsig(k) are summarized from median p-values with features called significant at alpha=0.05 using the Wilcoxon rank sum test
- The fully mixed scenario yields a low significant-feature count consistent with chance-level discoveries at alpha=0.05

## Limitations

- No formal inference method is provided; the AOI and AUMC measures are descriptive, not inferential
- The method does not control the false discovery rate or calculate adjusted p-values; FDR adjustment via empirical Bayes is only planned future work
- Ordering of signals by fragility index is not perfect, although simulations show high accuracy in identifying differential features
- The Wilcoxon-based version achieved higher accuracy than the DESeq-based version on simulated data, so alternate test choices may degrade performance

## Evidence

- [abstract] the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing."
- [methods] bin_permute_all invocation with parameters: "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [methods] plot_bin_psig invocation saving the U-curve: "sigloc<-plot_bin_psig(alloutputs=results,psigtitle=NULL,psigyrange=c(0,170),savepsigfile ="sigfeatures.eps""
- [abstract] Wilcoxon default test rationale: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [readme] README: top_pm constraint and run instruction: ">run progressive permutation. "variable" denotes the name of binary outcome. "top_pm" must be less than the number of microbial features"
