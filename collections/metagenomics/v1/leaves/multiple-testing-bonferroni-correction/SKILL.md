---
name: multiple-testing-bonferroni-correction
description: Use when many raw permutational ANOVA P-values are produced from pairwise
  treatment-group contrasts (e.g. LEVO vs OTHERS, CLA vs OTHERS across time points)
  and adjusted P-values must be reported to avoid false-positive group differences.
when_to_use_negative:
- The P-values are already multiple-testing adjusted (e.g. previously Bonferroni-
  or FDR-corrected), so a second correction would over-conservatively inflate them
- There is a single pre-planned comparison rather than a family of pairwise contrasts
  (LEVO/CLA vs OTHERS across time points), making correction unnecessary
- The input is upstream raw variant detection (FASTQ/BAM pileups from cellSNP-lite)
  — variant flagging is MetaQuad's job, not the correction step
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MetaQuad
  role: Flags the shared informative variants (num_of_clusters >= 2, recorded in BIC_params.csv)
    that define the test set of ARG allele-frequency profiles entering the pairwise
    contrasts
  repo: https://github.com/holab-hku/MetaQuad.git
- name: cellsnp-lite
  role: Produces the cellSNP.cells_ARG.vcf.gz allele pileup matrix from which the
    allele-frequency profiles are extracted before testing
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Python
  role: Runs permutational multivariate ANOVA / multilevel pairwise.adonis on the
    Bray-Curtis distances and performs the Bonferroni adjustment of the raw P-values
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:e3217cd9ed7400d23165cae56e5a780657c9cde87bacb6034594dae81f8072ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad.git
  tools:
  - MetaQuad
  - Python
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiple-testing-bonferroni-correction

## Summary

Applies Bonferroni correction to raw P-values from multilevel pairwise.adonis (permutational multivariate ANOVA) tests so that group-differentiation claims across antibiotic treatment arms and time points (e.g. LEVO vs OTHERS, CLA vs OTHERS at T+6W) retain family-wise error control.

## When to use

Use when many raw permutational ANOVA P-values are produced from pairwise treatment-group contrasts (e.g. LEVO vs OTHERS, CLA vs OTHERS across time points) and adjusted P-values must be reported to avoid false-positive group differences.

## When NOT to use

- The P-values are already multiple-testing adjusted (e.g. previously Bonferroni- or FDR-corrected), so a second correction would over-conservatively inflate them
- There is a single pre-planned comparison rather than a family of pairwise contrasts (LEVO/CLA vs OTHERS across time points), making correction unnecessary
- The input is upstream raw variant detection (FASTQ/BAM pileups from cellSNP-lite) — variant flagging is MetaQuad's job, not the correction step

## Inputs

- ARG allele-frequency matrix for the T+6W samples from cellSNP.cells_ARG.vcf.gz, restricted to MetaQuad shared informative variants (num_of_clusters >= 2 in BIC_params.csv)
- Treatment group labels per sample (CLA, LEVO, OTHERS) and time point
- Raw P-values from multilevel pairwise.adonis tests over the Bray-Curtis distance matrix

## Outputs

- Bonferroni-adjusted P-values table written to pairwise_adonis_ARG_T6W.csv (adjusted P = 0.03 LEVO vs OTHERS, 0.003 CLA vs OTHERS at T+6W; non-significant elsewhere)
- List of significant treatment-group contrasts per time point at the Bonferroni-adjusted alpha

## How to apply

First restrict the allele-frequency matrix to shared informative variants flagged by MetaQuad with num_of_clusters >= 2 in BIC_params.csv, so the tested profiles are the article's ARG allele frequencies from cellSNP.cells_ARG.vcf.gz rather than noise. Compute pairwise Bray-Curtis dissimilarities between samples and run permutational multivariate ANOVA with multilevel pairwise.adonis across the treatment groups (LEVO vs OTHERS, CLA vs OTHERS) at each time point. Collect the raw P-values from the full family of contrasts and Bonferroni-adjust them (multiply by the number of tests, capping at 1) so that family-wise error is controlled across time points and group contrasts. Write the Bonferroni-adjusted P-value table to pairwise_adonis_ARG_T6W.csv; in this study the adjustment confirmed adjusted P = 0.03 for LEVO vs OTHERS and P = 0.003 for CLA vs OTHERS at T+6W, with no comparison significant at other time points. Judge correctness by checking that the adjusted table preserves all raw comparisons, each adjusted P is >= its raw P, and the significant T+6W contrasts match the published values.

## Related tools

- **MetaQuad** (Flags the shared informative variants (num_of_clusters >= 2, recorded in BIC_params.csv) that define the test set of ARG allele-frequency profiles entering the pairwise contrasts) — https://github.com/holab-hku/MetaQuad.git
- **cellsnp-lite** (Produces the cellSNP.cells_ARG.vcf.gz allele pileup matrix from which the allele-frequency profiles are extracted before testing) — https://github.com/single-cell-genetics/cellsnp-lite
- **Python** (Runs permutational multivariate ANOVA / multilevel pairwise.adonis on the Bray-Curtis distances and performs the Bonferroni adjustment of the raw P-values)

## Examples

```
from statsmodels.stats.multitest import multipletests; adj_p = multipletests(raw_pairwise_adonis_p, method='bonferroni')[1]; pd.DataFrame({'comparison': comps, 'adj_p': adj_p}).to_csv('pairwise_adonis_ARG_T6W.csv')
```

## Evaluation signals

- pairwise_adonis_ARG_T6W.csv reproduces adjusted P = 0.03 for LEVO vs OTHERS and P = 0.003 for CLA vs OTHERS at T+6W
- No comparison is significant at any time point other than T+6W after adjustment
- The number of adjusted P-values equals the number of raw pairwise.adonis comparisons (no tests silently dropped from the family)
- Every Bonferroni-adjusted P is >= its raw P, and adjusted values are capped at 1
- The tested profiles contain only MetaQuad-flagged variants with num_of_clusters >= 2 from BIC_params.csv

## Limitations

- Bonferroni is conservative: it controls family-wise error at the cost of power, which contributes to no comparison reaching significance at time points other than T+6W
- Permutational P-value resolution is bounded by the permutation count (e.g. an adjusted P of 0.003 implies sufficiently many permutations); too few permutations make small adjusted P-values unreliable
- Results depend on the MetaQuad variant-flagging step — false positives or false negatives in shared informative variant detection propagate into the tested profiles and their P-values

## Evidence

- [results] At T+6W, multilevel pairwise comparison showed significant differences for LEVO versus OTHER (adjusted P = 0.03) and CLA versus OTHER (adjusted P = 0.003), whereas no comparison was significant at the other time points.: [evidence span withheld — anchor status: not_found]
- [readme] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [intro] Pile up variants using cellsnp-lite with recommended parameters minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
