---
name: microbiome-pvalue-trace-interpretation
description: Use when a progressive permutation run (bin_permute_all output on a binary
  grouping outcome) needs interpretation via U-curves and p-value traces to assess
  overall microbiome association strength.
when_to_use_negative:
- The grouping outcome is continuous or multi-class rather than binary — the bin_*
  functions of ProgPermute expect a binary outcome ('variable' denotes the name of
  binary outcome)
- The user needs formal inference or FDR-adjusted p-values — the paper states the
  method 'does not come with a formal inference method' and does not control the false
  discovery rate
- The question is an omnibus group-comparison test (e.g. PERMANOVA/PCoA ordination)
  rather than interpretation of progressive-permutation robustness curves
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ProgPermute
  role: R package supplying bin_permute_all, plot_bin_psig, plot_bin_pv, and plot_bin_permute_sigcurve
    used to generate and interpret the U-curve and p-value traces
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Runtime environment; DESeq2 package in R is used when p-values from DESeq
    are substituted for the default Wilcoxon test
- name: DESeq
  role: Alternative p-value source for the progressive permutation scenarios when
    the method argument is set to DESeq instead of the Wilcoxon rank sum test
- name: R-shiny
  role: User-facing web interface (ProgPerm ShinyApp) offering the same visualizations
    interactively
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:33ad1a019a0a256a2b7475468d1875b0090fd2d4b6d309dd0e0584483284265b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# microbiome-pvalue-trace-interpretation

## Summary

Interprets ProgPermute progressive-permutation outputs by reading the U-curve of the number of significant features versus proportion of mixing and the per-feature p-value traces to judge the strength of the microbiome–grouping-factor association and the robustness of individual hits.

## When to use

Use when a progressive permutation run (bin_permute_all output on a binary grouping outcome) needs interpretation via U-curves and p-value traces to assess overall microbiome association strength.

## When NOT to use

- The grouping outcome is continuous or multi-class rather than binary — the bin_* functions of ProgPermute expect a binary outcome ('variable' denotes the name of binary outcome)
- The user needs formal inference or FDR-adjusted p-values — the paper states the method 'does not come with a formal inference method' and does not control the false discovery rate
- The question is an omnibus group-comparison test (e.g. PERMANOVA/PCoA ordination) rather than interpretation of progressive-permutation robustness curves

## Inputs

- Binary grouping outcome column (e.g. 'variable') within a microbiome feature table such as testdata1 (267 microbial features)
- bin_permute_all output object (alloutputs), computed with Wilcoxon rank sum or DESeq p-values
- Parameters: top_pm (must be less than the number of microbial features), zoomn, alpha=0.05, samsize equal to the sample/feature size used for the scaled curve

## Outputs

- U-curve figure of the number of significant features versus proportion of mixing, saved as EPS (e.g. sigfeatures.eps via plot_bin_psig)
- Per-feature p-value traces across permutation scenarios, saved as EPS (e.g. pvfeatures.eps via plot_bin_pv)
- Scaled U-curve of the proportion of significant features, saved as EPS (e.g. bin_sigcurve.eps via plot_bin_permute_sigcurve)

## How to apply

Run bin_permute_all on the microbiome feature table (e.g. testdata1 with 267 features and the binary outcome 'variable') with top_pm set below the feature count (267), zoomn=15, and alpha=0.05; by default Wilcoxon rank sum p-values are computed, but the method argument can be set to DESeq so DESeq2 in R supplies the p-values. Then call plot_bin_psig with a psigyrange spanning the observed-to-full-permutation counts to render the U-curve of significant features versus proportion of mixing, plot_bin_pv (top_pm=267) to display the p-value traces of the top features, and plot_bin_permute_sigcurve (samsize=267) for the scaled U-curve, saving each as an EPS file. Interpretation relies on the U-curve shape: a decrease in significant features from the observed data (k=0) toward the fully mixed scenario k=K_f signals true group association, while a near-flat curve indicates little or no signal; individual hits are recommended by comparing their observed p-values with their p-values under full permutation. The resulting shape conveys the strength of the overall association and can be compared across grouping factors, but measures such as AOI or AUMC are descriptive rather than inferential.

## Related tools

- **ProgPermute** (R package supplying bin_permute_all, plot_bin_psig, plot_bin_pv, and plot_bin_permute_sigcurve used to generate and interpret the U-curve and p-value traces) — https://github.com/LyonsZhang/ProgPermute
- **R** (Runtime environment; DESeq2 package in R is used when p-values from DESeq are substituted for the default Wilcoxon test)
- **DESeq** (Alternative p-value source for the progressive permutation scenarios when the method argument is set to DESeq instead of the Wilcoxon rank sum test)
- **R-shiny** (User-facing web interface (ProgPerm ShinyApp) offering the same visualizations interactively)

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); sigloc<-plot_bin_psig(alloutputs=results,psigyrange=c(0,170),savepsigfile="sigfeatures.eps")
```

## Evaluation signals

- The rendered U-curve of significant features decreases from the observed data (k=0) toward the full permutation scenario k=K_f; a near-flat curve is expected only if no group signal exists
- All requested EPS output files (e.g. sigfeatures.eps, pvfeatures.eps, bin_sigcurve.eps) exist after the plotting calls
- top_pm passed to bin_permute_all is strictly less than the number of microbial features (267 for testdata1)
- plot_bin_psig returns the significant-feature locations object (e.g. sigloc) without error and the psigyrange spans the observed-to-full-permutation count range
- If DESeq is substituted, the same decreasing trend is reproduced, noting the paper reports the Wilcoxon version achieves higher accuracy in simulations

## Limitations

- The method does not come with a formal inference method; measures such as AOI or AUMC are descriptive, not inferential
- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR control is only planned future work
- On simulated data, the Wilcoxon-based version achieves higher accuracy than the DESeq-based version, so substituted p-value sources may be less accurate
- Ordering of signals by fragility index is not perfect

## Evidence

- [abstract] the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength"
- [methods] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [readme] >run progressive permutation. "variable" denotes the name of binary outcome. "top_pm" must be less than the number of microbial features: ">run progressive permutation. "variable" denotes the name of binary outcome. "top_pm" must be less than the number of microbial features"
- [discussion] The measures we propose, such as AOI or AUMC, are meant to be descriptive: "The measures we propose, such as AOI or AUMC, are meant to be descriptive"
- [methods] the version based on the Wilcoxon test achieves higher accuracy than the version based on DESeq: "the version based on the Wilcoxon test achieves higher accuracy than the version based on DESeq"
