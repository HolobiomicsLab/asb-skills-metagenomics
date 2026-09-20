---
name: bonferroni-multiple-testing-correction
description: Use when comparing abundances of many predicted functional modules (e.g.
  KEGG modules M00009, M00240) between sample groups and need family-wise error control
  over the resulting battery of statistical tests.
when_to_use_negative:
- When only one pre-specified hypothesis test is performed — Bonferroni correction
  is unnecessary and only reduces power
- When OTU/microbiome data are taxonomic profiles rather than predicted functional
  module abundances, since the tested quantities differ
- When exploratory ranking (not confirmatory significance) is the goal and family-wise
  error control would discard true weak signals
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: STAMP
  role: Statistical software used to run Welch's t-tests between body-site groups
    and apply Bonferroni correction to module-level p-values
- name: PICRUSt
  role: Produces the predicted metagenome (KO/module abundances) whose between-group
    differences are tested with corrected statistics
  repo: http://picrust.github.com
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:257300083613141d856486da023e4c5d1723e99aefc5890d806d4aa2b09a8753
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PICRUSt
  - QIIME
  - STAMP
  - uclust
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# bonferroni-multiple-testing-correction

## Summary

Apply Bonferroni correction to control the family-wise error rate when testing many KEGG modules (or other functional categories) for differential abundance across body sites or time points, as done for PICRUSt-predicted HMP metagenomes in STAMP.

## When to use

Use when comparing abundances of many predicted functional modules (e.g. KEGG modules M00009, M00240) between sample groups and need family-wise error control over the resulting battery of statistical tests.

## When NOT to use

- When only one pre-specified hypothesis test is performed — Bonferroni correction is unnecessary and only reduces power
- When OTU/microbiome data are taxonomic profiles rather than predicted functional module abundances, since the tested quantities differ
- When exploratory ranking (not confirmatory significance) is the goal and family-wise error control would discard true weak signals

## Inputs

- Table of KEGG module (or KO) abundances per sample, collapsed from PICRUSt-predicted metagenomes
- Sample-to-group metadata mapping (e.g. supragingival vs. subgingival plaque; menses vs. non-menses time points)
- List of modules/tests to correct (defines the Bonferroni multiplier)

## Outputs

- Per-module Welch's t-test statistics with Bonferroni-corrected p-values (e.g. pathway_enrichment_results.tsv)
- Rank ordering of modules by significance/enrichment
- Significance calls (e.g. M00009 corrected p<1e-10 for supragingival plaque enrichment)

## How to apply

After collapsing PICRUSt-predicted KO abundances to KEGG module level, test each module of interest between groups (e.g. supragingival vs. subgingival plaque from the 6,431-sample HMP OTU table) using Welch's t-test, which does not assume equal variances. Multiply each raw p-value by the number of tests performed (Bonferroni correction) to obtain family-wise-corrected p-values, judging significance at the corrected threshold (e.g. citrate cycle M00009 enrichment in supragingival plaque at corrected p<1e-10). In practice this was executed in STAMP v2.0, which applies the correction and produces the adjusted statistics and visualizations. The same corrected-testing scheme is used for time-course contrasts, e.g. testing whether M00240 iron complex transport system abundance increases at menses time points versus the rest of the vaginal cycle. Record test statistics, raw and corrected p-values, and rank orderings so that correction is transparent and reproducible.

## Related tools

- **STAMP** (Statistical software used to run Welch's t-tests between body-site groups and apply Bonferroni correction to module-level p-values)
- **PICRUSt** (Produces the predicted metagenome (KO/module abundances) whose between-group differences are tested with corrected statistics) — http://picrust.github.com

## Examples

```
collapse KOs to KEGG modules with PICRUSt, then in STAMP v2.0 run Welch's t-test with Bonferroni correction on module abundances (supragingival vs. subgingival plaque) and write pathway_enrichment_results.tsv
```

## Evaluation signals

- Corrected p-value for citrate cycle (M00009) supragingival-vs-subgingival comparison is significant at p<1e-10 (Welch's t-test with Bonferroni correction)
- M00240 iron complex transport system shows the largest significant mean-abundance increase during menses after correction
- Output table (pathway_enrichment_results.tsv) contains test statistics, p-values, and rank orderings for every tested module
- Bonferroni-adjusted p-values equal raw p-values multiplied by the number of tests and never exceed 1.0
- NSTI values recorded per sample so low-confidence predictions can be excluded before testing

## Limitations

- Bonferroni correction is conservative; with large test batteries (thousands of KOs/modules) true weak effects may fail the corrected threshold
- Predictions depend on reference genome availability (NSTI); some gain/loss or laterally transferred genes will be poorly predicted, adding uncertainty to the tested abundances
- PICRUSt only infers bacterial and archaeal components; viral or eukaryotic contributions are absent from the tested profiles

## Evidence

- [results] enrichment of citrate cycle (M00009) genes in supragingival versus subgingival plaque (p<1e-10, Welch's t-test with Bonferroni correction): [evidence span withheld — anchor status: not_found]
- [other] Compare module abundances between supragingival and subgingival plaque samples using Welch's t-test with Bonferroni correction in STAMP v2.0: [evidence span withheld — anchor status: not_found]
