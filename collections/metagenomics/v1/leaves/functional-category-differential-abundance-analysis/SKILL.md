---
name: functional-category-differential-abundance-analysis
description: Use when aggregated functional-category summary files exist for multiple
  metatranscriptome samples in control vs. experimental groups and you need to identify
  significantly differing functions with adjusted p-values and fold changes.
when_to_use_negative:
- Input is raw FASTQ reads or per-read annotation output rather than aggregated functional
  summary tables — run aggregation first
- You need organism-level (taxonomic) rather than functional-category significance
  testing — use R_organism_script.Rmd instead
- Input features are already reduced to a short list (no long tail); thresholding
  and low-abundance filtering are unnecessary and could discard real signal
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0203
tools:
- name: R_functional_script_w-md5s.Rmd
  role: Imports functional category summary files and performs statistical significance
    correlations with DESeq2
  repo: https://github.com/transcript/SAMSA
- name: analysis_counter_w-md5s.py
  role: Creates sorted functional abundance summaries that retain the m5nr ID from
    MG-RAST for transcript identification
  repo: https://github.com/transcript/SAMSA
- name: long_tail_threshold.py
  role: Thresholds summary files at a chosen cutoff percentage to remove low-abundance
    long-tail noise before DESeq2 testing
  repo: https://github.com/transcript/SAMSA
- name: md5-R-results_query.py
  role: Submits MG-RAST internal IDs from the functional transcript results to MG-RAST's
    API to determine RefSeq IDs
  repo: https://github.com/transcript/SAMSA
- name: DESeq2
  role: Negative-binomial differential expression model used within the R functional
    script to call significant functional categories
- name: R
  role: Runtime environment (preferably RStudio) for the functional significance analysis
    scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:8fa030c2c72a9a1465f3635c5df237f6c3e4efc677973f2fbcce6c78b8c23fe1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - DESeq2
  - R
  - R_functional_script_w-md5s.Rmd
  - SAMSA
  - analysis_counter_w-md5s.py
  - long_tail_threshold.py
  - md5-R-results_query.py
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# functional-category-differential-abundance-analysis

## Summary

Statistical comparison of functional annotation categories (e.g. SEED Subsystems or MG-RAST functional summaries) between control and experimental metatranscriptome groups using DESeq2 negative-binomial modeling, as implemented in the SAMSA pipeline's R_functional_script_w-md5s.Rmd.

## When to use

Use when aggregated functional-category summary files exist for multiple metatranscriptome samples in control vs. experimental groups and you need to identify significantly differing functions with adjusted p-values and fold changes.

## When NOT to use

- Input is raw FASTQ reads or per-read annotation output rather than aggregated functional summary tables — run aggregation first
- You need organism-level (taxonomic) rather than functional-category significance testing — use R_organism_script.Rmd instead
- Input features are already reduced to a short list (no long tail); thresholding and low-abundance filtering are unnecessary and could discard real signal

## Inputs

- Aggregated functional category summary files (from analysis_counter_w-md5s.py, retaining MG-RAST m5nr IDs)
- Sample metadata designating control vs. experimental groups (e.g. 9 Tyk2−/− vs. 6 wild-type)
- Optionally, MG-RAST annotation results downloaded via MG-RAST_API_downloader.py

## Outputs

- Tabular file of significantly differing functional categories with p-values, adjusted p-values, and log2FoldChange (significance cutoff: adjusted p < 0.05)
- Optionally, RefSeq IDs for significant transcripts retrieved via md5-R-results_query.py from MG-RAST's API

## How to apply

Import the functional category summary files into R (preferably RStudio) using R_functional_script_w-md5s.Rmd, which pairs with analysis_counter_w-md5s.py so that the MG-RAST m5nr IDs are retained for later transcript identification. Optionally run long_tail_threshold.py first to threshold summary files at a chosen cutoff percentage and remove low-abundance long-tail noise before testing. Normalize read counts and fit the DESeq2 negative-binomial model comparing the two groups (e.g. 9 Tyk2−/− samples vs. 6 wild-type controls). Correct for multiple testing and extract categories/transcripts reaching adjusted p < 0.05, writing tabular output with p-values and log2FoldChange scores. Use md5-R-results_query.py to submit retained MG-RAST internal IDs to MG-RAST's API when RefSeq IDs for significant transcripts are needed.

## Related tools

- **R_functional_script_w-md5s.Rmd** (Imports functional category summary files and performs statistical significance correlations with DESeq2) — https://github.com/transcript/SAMSA
- **analysis_counter_w-md5s.py** (Creates sorted functional abundance summaries that retain the m5nr ID from MG-RAST for transcript identification) — https://github.com/transcript/SAMSA
- **long_tail_threshold.py** (Thresholds summary files at a chosen cutoff percentage to remove low-abundance long-tail noise before DESeq2 testing) — https://github.com/transcript/SAMSA
- **md5-R-results_query.py** (Submits MG-RAST internal IDs from the functional transcript results to MG-RAST's API to determine RefSeq IDs) — https://github.com/transcript/SAMSA
- **DESeq2** (Negative-binomial differential expression model used within the R functional script to call significant functional categories)
- **R** (Runtime environment (preferably RStudio) for the functional significance analysis scripts)

## Examples

```
Rscript -e "rmarkdown::render('R_functional_script_w-md5s.Rmd')"  # after aggregation with analysis_counter_w-md5s.py and optional long_tail_threshold.py cutoff
```

## Evaluation signals

- Adjusted p-values are computed after multiple-testing correction and only features with adjusted p < 0.05 are reported as significant
- The output table includes both assigned p-values and log2FoldChange scores for each functional feature
- m5nr IDs are preserved through analysis_counter_w-md5s.py so significant transcripts can be resolved to RefSeq IDs via md5-R-results_query.py
- On the Tyk2 knockout vs. wild-type DSS colitis benchmark, the pipeline should recover the reported >300 differentially expressed protein-coding transcripts at the significance cutoff
- Low-abundance long-tail categories are removed (or demonstrably excluded) when long_tail_threshold.py is applied before testing

## Limitations

- MG-RAST's annotation servers require waiting in a queue, and processing may be slowed by days to weeks, delaying the functional summaries this analysis depends on
- mRNA read depth must be sufficient: each sample needs ~40-50 million raw sequences to yield the 5-10 million annotated reads necessary for accurate abundance measures; insufficient annotated mRNA reads undermine significance calls
- In SAMSA v1, functional analysis via MG-RAST is not tailored to eukaryotic organisms, which can increase mismatch rates

## Evidence

- [methods] R_functional_script_w-md5s.Rmd - imports functional category summary files and performs statistical significance correlations: "R_functional_script_w-md5s.Rmd - imports functional category summary files and performs statistical significance correlations"
- [methods] long_tail_threshold.py - permits thresholding of summary files at a chosen cutoff percentage: "long_tail_threshold.py - permits thresholding of summary files at a chosen cutoff percentage"
- [methods] md5-R-results_query.py - submits MG-RAST internal IDs for a functional transcript file to MG-RAST's API: "md5-R-results_query.py - submits MG-RAST internal IDs for a functional transcript file to MG-RAST's API"
- [abstract] We were also able to detect significant differences between control and experimental groups in both organism transcriptional activity and specific cellular functions.: "We were also able to detect significant differences between control and experimental groups in both organism transcriptional activity and specific cellular functions."
- [readme] R scripts use DESeq to compute most significantly different features between control vs. experimental samples.: [evidence span withheld — anchor status: external_doc]
- [abstract] Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures.: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
