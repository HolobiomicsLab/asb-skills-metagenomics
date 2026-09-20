---
name: continuous-outcome-progressive-permutation
description: Use when the grouping/clinical variable is continuous (not binary) and
  you want to gauge the overall strength of the microbiome-outcome association and
  rank feature robustness via a progressive-permutation U-curve.
when_to_use_negative:
- The grouping outcome is binary or categorical — use the binary driver bin_permute_all
  with the default Wilcoxon rank sum test instead of con_permute_all.
- You need formal inference, FDR-controlled or adjusted p-values — the paper states
  the method is for exploration/visualization and 'does not control the false discovery
  rate or calculate adjusted p-values'.
- top_pm is not smaller than the number of microbial features — the README notes "top_pm"
  must be less than the number of microbial features.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: ProgPermute
  role: R package providing the progressive permutation machinery and the continuous-outcome
    driver con_permute_all over scenarios k=0..Kf
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Implementation environment in which data(testdata1) is loaded, con_permute_all
    is executed, and Kendall's tau / Spearman rank correlation tests are computed
    per feature
- name: R-shiny
  role: Interactive web front end with visualizations of the U-curve and discovery
    robustness (https://biostatistics.mdanderson.org/shinyapps/ProgPerm)
  repo: https://biostatistics.mdanderson.org/shinyapps/ProgPerm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:72411f1d4344a0f0557e3a695975bdbe67051409f024d1301eb9a64047c14ac0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# continuous-outcome-progressive-permutation

## Summary

Extension of the ProgPermute progressive permutation framework to a continuous grouping outcome: instead of permuting binary labels and testing with Wilcoxon, a proportion of the continuous outcome is permuted per scenario and per-feature significance is assessed with Kendall's tau and Spearman rank correlation tests, yielding a scaled U-curve of significant-feature counts versus mixing proportion.

## When to use

Use when the grouping/clinical variable is continuous (not binary) and you want to gauge the overall strength of the microbiome-outcome association and rank feature robustness via a progressive-permutation U-curve.

## When NOT to use

- The grouping outcome is binary or categorical — use the binary driver bin_permute_all with the default Wilcoxon rank sum test instead of con_permute_all.
- You need formal inference, FDR-controlled or adjusted p-values — the paper states the method is for exploration/visualization and 'does not control the false discovery rate or calculate adjusted p-values'.
- top_pm is not smaller than the number of microbial features — the README notes "top_pm" must be less than the number of microbial features.

## Inputs

- R data object of microbiome feature abundances with a continuous grouping outcome column (e.g. packaged testdata1 from the ProgPermute package)
- Name of the continuous grouping variable (argument 'variable')
- top_pm: number of top features to track (must be less than the number of microbial features)
- zoomn: zoom-in parameter for per-feature traces
- alpha: significance cutoff (default 0.05)

## Outputs

- U-curve summary table written to ucurve_summary_table.csv (proportion of mixing vs. significant-feature counts/proportions for Kendall's tau and Spearman tests)
- Per-scenario significant-feature counts nsig(k) scaled by total features p (nsig(k)/p in [0,1])
- Per-scenario median per-feature p-values for the tracked top features
- U-curve visualization of significant features versus proportion of mixing

## How to apply

Load the packaged sample data in R with data(testdata1) from the ProgPermute package (install via install_github("LyonsZhang/ProgPermute")). Run the continuous-outcome driver con_permute_all(variable, testdata, top_pm, zoomn, alpha=0.05) over scenarios k=0 (observed data) through the full permutation scenario Kf = ceil((n1*n2-1)/(n1+n2+2)), permuting a proportion of the continuous grouping outcome at each scenario rather than swapping binary labels. At each scenario, compute per-feature p-values with Kendall's tau and Spearman rank correlation tests, summarize the draws per scenario by median p-values, and count significant features nsig(k) = #{j : p_j^m(k) <= 0.05}. Scale counts by the total number of features (nsig(k)/p, ranging 0 to 1) so U-curves from different datasets are comparable, then write the summary table (proportion of mixing vs. significant-feature counts/proportions for both correlation tests) to ucurve_summary_table.csv. A deeper, steeper U-curve (high nsig at k=0, minimum at full mixing) conveys stronger overall microbiome-grouping association; top hits whose p-value traces stay low across scenarios are the robust discoveries.

## Related tools

- **ProgPermute** (R package providing the progressive permutation machinery and the continuous-outcome driver con_permute_all over scenarios k=0..Kf) — https://github.com/LyonsZhang/ProgPermute
- **R** (Implementation environment in which data(testdata1) is loaded, con_permute_all is executed, and Kendall's tau / Spearman rank correlation tests are computed per feature)
- **R-shiny** (Interactive web front end with visualizations of the U-curve and discovery robustness (https://biostatistics.mdanderson.org/shinyapps/ProgPerm)) — https://biostatistics.mdanderson.org/shinyapps/ProgPerm

## Examples

```
library(ProgPermute); data(testdata1); results <- con_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05)
```

## Evaluation signals

- The resulting plot of significant features versus proportion of mixing forms a U-curve: high counts at k=0 (observed data), minimum near the fully mixed scenario Kf = ceil((n1*n2-1)/(n1+n2+2)).
- Scaled significant-feature proportions nsig(k)/p all fall within [0,1] across scenarios.
- Significance is judged at alpha = 0.05 per feature per scenario; per-scenario summaries are medians over permutation draws.
- Both correlation tracks (Kendall's tau and Spearman rank correlation) are reported and written to ucurve_summary_table.csv.
- top_pm used is strictly less than the total number of microbial features p, per the README constraint.

## Limitations

- The method is designed for general exploration and visualization and comes with no formal inference method; AOI/AUMC-style measures are descriptive, not inferential.
- It does not control the false discovery rate or calculate adjusted p-values; empirical-Bayes FDR control is only planned future work.
- Ordering of signals by robustness is not perfect, though simulations show high accuracy for identifying differential features.
- Default binary-outcome testing uses the Wilcoxon rank sum test; the continuous-outcome extension instead relies on Kendall's tau and Spearman rank correlation tests, so results are not directly comparable to the bin_* Wilcoxon pipeline.

## Evidence

- [abstract] U-curve of significant features versus mixing: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing. The shape of the U-Curve can convey the strength of the overall association"
- [abstract] Progressive permutation of grouping labels: "we progressively permute the grouping factor labels of the microbiome samples and perform multiple differential abundance tests in each scenario."
- [methods] Default nonparametric testing rationale: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [other] Continuous-outcome driver and scenario range: [evidence span withheld — anchor status: not_found]
- [other] Correlation tests and median summarization: [evidence span withheld — anchor status: not_found]
- [readme] What the method evaluates: "The method can evaluate the overall association between microbiome and the grouping factor, rank the robustness of the discovered microbes"
