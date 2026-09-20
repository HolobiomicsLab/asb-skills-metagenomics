---
name: descriptive-association-metric-quantification
description: Use when multiple binary clinical/grouping factors are available for
  the same microbiome feature table and the goal is a quantitative, comparable measure
  (AUMC/AOI) of which factor identifies differences in microbial communities.
when_to_use_negative:
- The question is formal inference (p-value adjustment or FDR control) — the paper
  states the method does not control the false discovery rate and AOI/AUMC are descriptive
  only
- The grouping factor is not binary (multi-class or continuous factors need regrouping
  first, as done for SmitsData 1/2)
- The goal is to validate individual feature hits robustly (use fragility index or
  the full-permutation best-feature workflow instead)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ProgPermute
  role: R package providing bin_permute_all and plot_bin_permute_sigcurve to run progressive
    permutation and draw the scaled U-curve used to compute the AUMC
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R-shiny
  role: User-friendly interface with visualizations of the U-curve and permutation
    results without direct R scripting
- name: R
  role: Implementation environment for ProgPermute, installed via devtools install_github('LyonsZhang/ProgPermute')
  repo: https://github.com/LyonsZhang/ProgPerm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:f4a33e171b1550ca4277170644dd5a947f913222b2ba1231cc4e38bbf46aa8bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# descriptive-association-metric-quantification

## Summary

Quantify the overall association between the microbiome and binary grouping factors by computing the AUMC (area under the U-curve of significant features vs. proportion of mixing) from progressive permutation, then rank factors by decreasing AUMC (bigger AUMC = stronger association).

## When to use

Use when multiple binary clinical/grouping factors are available for the same microbiome feature table and the goal is a quantitative, comparable measure (AUMC/AOI) of which factor best identifies differences in microbial communities.

## When NOT to use

- The question is formal inference (p-value adjustment or FDR control) — the paper states the method does not control the false discovery rate and AOI/AUMC are descriptive only
- The grouping factor is not binary (multi-class or continuous factors need regrouping first, as done for SmitsData 1/2)
- The goal is to validate individual feature hits robustly (use fragility index or the full-permutation best-feature workflow instead)

## Inputs

- Microbiome feature (OTU) count table with sample metadata containing binary grouping factors (e.g. testdata1, SmitsData 1/2, DeFilippo Data)
- Binary grouping factor variable name (e.g. 'variable') to be progressively permuted
- alpha significance threshold (0.05) and top_pm (top differentially abundant features, must be less than the number of microbial features)

## Outputs

- AUMC value per grouping factor (descriptive area under the U-curve, bounded in [0,1])
- Ranked table of grouping factors by decreasing AUMC (e.g. aumc_ranking.csv with factor, AUMC, rank)
- U-curve of nsig(k)/p versus proportion of mixing (e.g. bin_sigcurve.eps from plot_bin_permute_sigcurve)

## How to apply

Load the feature table and grouping factor (e.g. ProgPermute's packaged testdata1) and run progressive permutation with bin_permute_all using alpha=0.05 and the default Wilcoxon rank sum test to obtain the number of significant features nsig(k) at each permutation scenario k. Scale nsig(k) by the total feature count p and plot against proportion of mixing k/K to form the U-curve, whose minimum lies at the full permutation scenario K_f = ceil((n1*n2-1)/(n1+n2+2)). Numerically integrate the scaled curve from the observed data (k=0) to K_f to obtain the AUMC, e.g. via the curve drawn by plot_bin_permute_sigcurve. Repeat for every factor of interest (e.g. location vs. gender in the DeFilippo data; Dry/Wet vs. LDEW/LWED in the Smits data), assemble a table of AUMC (and optionally AOI) per factor, and rank factors in decreasing AUMC order since a larger AUMC indicates a stronger overall association.

## Related tools

- **ProgPermute** (R package providing bin_permute_all and plot_bin_permute_sigcurve to run progressive permutation and draw the scaled U-curve used to compute the AUMC) — https://github.com/LyonsZhang/ProgPermute
- **R-shiny** (User-friendly interface with visualizations of the U-curve and permutation results without direct R scripting)
- **R** (Implementation environment for ProgPermute, installed via devtools install_github('LyonsZhang/ProgPermute')) — https://github.com/LyonsZhang/ProgPerm

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); sigsum<-plot_bin_permute_sigcurve(alloutputs=results,samsize=267,lgndcol=2,savepsigfile="bin_sigcurve.eps")
```

## Evaluation signals

- AUMC values must lie between 0 and 1 since they are areas under the curve of nsig(k)/p
- AUMC should be near 0 (near-flat U-curve) when no true group difference exists
- Results should reproduce reported orderings, e.g. location AUMC 0.253 > gender AUMC 0.035 in the DeFilippo data
- The full permutation scenario K_f must equal ceil((n1*n2-1)/(n1+n2+2)) and the U-curve minimum should occur there
- top_pm passed to bin_permute_all must be less than the number of microbial features in the input table

## Limitations

- AUMC and AOI are descriptive measures only; the method does not come with a formal inference method
- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR control is only planned future work
- Designed mainly for general exploration and visualization of microbiome data rather than confirmatory testing

## Evidence

- [abstract] U-curve conveys overall association strength: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength of the overall association"
- [methods] Wilcoxon default test: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [readme] Concrete bin_permute_all invocation from README: "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [discussion] Descriptive-only caveat from discussion: "The measures we propose, such as AOI or AUMC, are meant to be descriptive"
- [intro] Comparison question motivation: "they do not answer a more general question as to which grouping factors better identify more differences in microbiome communities"
