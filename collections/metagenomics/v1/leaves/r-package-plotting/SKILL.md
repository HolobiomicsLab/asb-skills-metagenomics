---
name: r-package-plotting
description: Use when you have run ProgPermute's bin_permute_all on a binary grouping
  factor and need to visualize whether top hits from the original data show a decreasing
  -log10 p-value trend across permutation scenarios.
when_to_use_negative:
- When you need a formal inferential test or adjusted p-values — the method is explicitly
  descriptive and does not control the false discovery rate or calculate adjusted
  p-values.
- When the goal is the overall association U-curve (number of significant features
  vs. proportion of mixing) rather than per-feature p-value traces — use plot_bin_psig
  or plot_bin_permute_sigcurve instead.
- When your p-values come from a different differential testing framework's own plotting
  and you have not run bin_permute_all to produce the alloutputs object.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: ProgPermute
  role: R package whose plot_bin_pv function generates the p-value trace plot from
    bin_permute_all results
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: runtime environment in which the ProgPermute package functions are executed
- name: R-shiny
  role: interactive web front-end exposing the same trace visualizations without scripting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:46afe85513093884d7233eaee25ed592f5576fccc234135a413bcec764038394
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# r-package-plotting

## Summary

Generate the p-value trace plot (plot_bin_pv) of ProgPermute's R package, which ranks the top differential microbiome features from the original data by observed p-value and plots -log10 median p-values with 95% quantile intervals across progressive permutation scenarios to visualize robustness of discoveries.

## When to use

Use when you have run ProgPermute's bin_permute_all on a binary grouping factor and need to visualize whether top hits from the original data show a decreasing -log10 p-value trend across permutation scenarios.

## When NOT to use

- When you need a formal inferential test or adjusted p-values — the method is explicitly descriptive and does not control the false discovery rate or calculate adjusted p-values.
- When the goal is the overall association U-curve (number of significant features vs. proportion of mixing) rather than per-feature p-value traces — use plot_bin_psig or plot_bin_permute_sigcurve instead.
- When your p-values come from a different differential testing framework's own plotting and you have not run bin_permute_all to produce the alloutputs object.

## Inputs

- alloutputs object returned by ProgPermute::bin_permute_all
- Microbiome feature-by-sample test dataset loaded as data(testdata1) with a binary grouping column named 'variable'
- top_pm parameter (number of top features to trace; must be less than the number of microbial features, e.g. 267)

## Outputs

- EPS figure file of p-value traces (e.g. pvfeatures.eps) with pvpicdim dimensions such as c(10,7)
- Visual traces of -log10 median p-values with 95% quantile intervals for top features across permutation scenarios
- Returned plotting object (e.g. pvloc) capturing plotted trace locations

## How to apply

After running progressive permutation with bin_permute_all(variable, testdata, top_pm, zoomn, alpha=0.05), call plot_bin_pv(alloutputs, top_pm, pvyrange, savepvfile, pvpicdim) on the returned alloutputs object. The function ranks the top_pm features by their observed p-value at k=0 (no permutation) and, for each permutation mixing scenario k from 0 toward the full permutation scenario, plots the -log10 median p-value with 95% quantile intervals as a trace. Judge success by checking that the saved EPS file exists and that traces of truly significant variables start high at k=0 (e.g. y-range up to about 7, i.e. p ~ 1e-7) and visually decrease toward nonsignificance as mixing approaches the full permutation; flat or non-decreasing traces suggest fragile or non-robust hits.

## Related tools

- **ProgPermute** (R package whose plot_bin_pv function generates the p-value trace plot from bin_permute_all results) — https://github.com/LyonsZhang/ProgPermute
- **R** (runtime environment in which the ProgPermute package functions are executed)
- **R-shiny** (interactive web front-end exposing the same trace visualizations without scripting)

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); pvloc<-plot_bin_pv(alloutputs=results,top_pm=267,pvyrange=c(0,7),savepvfile="pvfeatures.eps",pvpicdim=c(10,7))
```

## Evaluation signals

- The output file pvfeatures.eps is created at the specified working path with pvpicdim dimensions.
- Traces of more significant variables start high at k=0 (no permutation) and decrease toward nonsignificance as mixing approaches the full permutation scenario, within pvyrange=c(0,7).
- top_pm is less than the total number of microbial features (README constraint, e.g. top_pm=267 for testdata1).
- Traces show 95% quantile intervals; true-positive features remain visually distinguishable from the permuted null across scenarios.
- Reproducibility check: rerunning Execute_ProgPermute.R in the repo's test folder reproduces the same decreasing trace pattern.

## Limitations

- Measures are descriptive only; the method does not control the false discovery rate or calculate adjusted p-values.
- No formal inference method is provided — AOI/AUMC-type summaries accompanying these plots are meant to be descriptive.
- Ordering of signals by robustness/fragility is not perfect, though simulations show high accuracy for identifying differential features.
- Wilcoxon rank sum is the default test; other methods (e.g. DESeq2) can be used but accuracy may differ (Wilcoxon-based version achieved higher accuracy on simulated data).

## Evidence

- [methods] Generate the p-value trace plot with plot_bin_pv(alloutputs=results, top_pm=267, pvyrange=c(0,7), savepvfile="pvfeatures.eps", pvpicdim=c(10,7)): [evidence span withheld — anchor status: not_found]
- [methods] verify the file pvfeatures.eps exists: [evidence span withheld — anchor status: not_found]
- [readme] README invocation of plot_bin_pv: ">show the traces of Pvalues
```R
pvloc<-plot_bin_pv(alloutputs=results,top_pm=267,pvtitle="",pvyrange=c(0,7),savepvfile ="pvfeatures.eps", pvpicdim=c(10,7))
```"
- [readme] README claim about decreasing trend: "We compare the signal strength of top hits from the original data with their performance in permutations, and will observe an apparent decreasing trend if these top hits are true positives"
- [discussion] Article limitation on FDR: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
