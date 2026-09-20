---
name: significant-feature-count-summarization
description: Use when you have run bin_permute_all on a binary-grouped microbiome
  feature table and need to aggregate per-scenario p-values into a significant-feature
  count curve (nsig(k)) to visualize overall association strength and validate that
  top hits are true positives.
when_to_use_negative:
- You need formal inference on individual features (adjusted p-values or FDR control)
  — the article states the method does not control the false discovery rate or calculate
  adjusted p-values
- The grouping factor is not binary — the bin_* family (bin_permute_all, plot_bin_psig)
  targets binary outcomes only
- You have raw OTU counts but have not yet run progressive permutation — plot_bin_psig
  requires the alloutputs object from bin_permute_all, not a feature table directly
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: ProgPermute
  role: R package providing bin_permute_all to progressively permute grouping labels
    and plot_bin_psig to render the significant-feature-count U-curve
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Runtime environment in which testdata1 is loaded and the summarization/plotting
    functions are executed
- name: R-shiny
  role: Interactive web interface for the same U-curve workflow without scripting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:ddcf402ef9f4c21a54ae74c9685bab78bc6bf0f00176bed040ac95946cba3c9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# significant-feature-count-summarization

## Summary

Summarizes the per-scenario outputs of ProgPermute's progressive permutation (per-scenario median p-values from Wilcoxon rank sum tests at alpha=0.05) into the count of significant taxa nsig(k) at each mixing level, and renders the characteristic U-curve of significant-feature count versus proportion of mixing via plot_bin_psig. The U-curve's depth and shape convey the strength of the overall microbiome–grouping association.

## When to use

Use when you have run bin_permute_all on a binary-grouped microbiome feature table and need to aggregate per-scenario p-values into a significant-feature count curve (nsig(k)) to visualize overall association strength and validate that top hits are true positives.

## When NOT to use

- You need formal inference on individual features (adjusted p-values or FDR control) — the article states the method does not control the false discovery rate or calculate adjusted p-values
- The grouping factor is not binary — the bin_* family (bin_permute_all, plot_bin_psig) targets binary outcomes only
- You have raw OTU counts but have not yet run progressive permutation — plot_bin_psig requires the alloutputs object from bin_permute_all, not a feature table directly

## Inputs

- Output object from bin_permute_all (per-scenario p-values for top_pm features, e.g. top_pm=267, zoomn=15, alpha=0.05)
- Microbiome feature-count table with a binary grouping factor (e.g. data(testdata1) from the ProgPermute package)
- Significance threshold alpha=0.05 and y-axis range (e.g. psigyrange=c(0,170))

## Outputs

- U-curve plot of number of significant features versus proportion of mixing, saved as EPS (e.g. sigfeatures.eps)
- Vector of significant-feature counts nsig(k) across progressive permutation scenarios
- Visual assessment of overall association strength (decreasing-then-U trend comparable to the deposited UCurve.png)

## How to apply

First run the progressive permutation with results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05), where top_pm must be smaller than the number of microbial features and 'variable' names the binary outcome column. Then summarize each scenario's per-feature p-values (default Wilcoxon rank sum test) into the count of significant taxa nsig(k) at each progressive permutation scenario. Call plot_bin_psig(alloutputs=results, psigyrange=c(0,170), savepsigfile="sigfeatures.eps") to draw the U-curve of significant-feature count versus proportion of mixing, saving as EPS. The expected shape is a decreasing trend from the observed data toward full permutation followed by the U-form; a pronounced, deeper U-curve indicates a stronger overall association between the microbiome and the grouping factor.

## Related tools

- **ProgPermute** (R package providing bin_permute_all to progressively permute grouping labels and plot_bin_psig to render the significant-feature-count U-curve) — https://github.com/LyonsZhang/ProgPermute
- **R** (Runtime environment in which testdata1 is loaded and the summarization/plotting functions are executed)
- **R-shiny** (Interactive web interface for the same U-curve workflow without scripting)

## Examples

```
sigloc<-plot_bin_psig(alloutputs=results,psigtitle=NULL,psigyrange=c(0,170),savepsigfile="sigfeatures.eps", psigpicdim=c(10,7))
```

## Evaluation signals

- sigfeatures.eps exists on disk and is a valid EPS file after plot_bin_psig completes
- The plotted curve shows a decreasing trend in significant-feature counts from the observed data toward full permutation before forming the U-shape
- The significant-feature count at the observed (unmixed) end falls within the specified y-range (e.g. psigyrange=c(0,170))
- The rendered curve is visually comparable to the deposited UCurve.png reference from the ProgPermute test folder
- All nsig(k) values are computed with the same alpha=0.05 cutoff across scenarios, and top_pm (267) is strictly less than the total number of microbial features

## Limitations

- The U-curve and related measures (AOI, AUMC) are descriptive, not inferential — no formal inference method is provided
- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR adjustment is only planned future work
- top_pm must be less than the number of microbial features; a curve based on too small a top_pm will not represent the full association signal

## Evidence

- [abstract] the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength of the overall association: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength of the overall association"
- [readme] results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05): "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [methods] sigloc<-plot_bin_psig(alloutputs=results,psigtitle=NULL,psigyrange=c(0,170),savepsigfile ="sigfeatures.eps": "sigloc<-plot_bin_psig(alloutputs=results,psigtitle=NULL,psigyrange=c(0,170),savepsigfile ="sigfeatures.eps""
- [abstract] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [discussion] At this time, our method does not control the false discovery rate or calculate adjusted p-values.: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
