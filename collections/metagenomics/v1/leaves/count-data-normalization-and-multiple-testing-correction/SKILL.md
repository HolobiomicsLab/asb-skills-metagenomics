---
name: count-data-normalization-and-multiple-testing-correction
description: Use when you have raw MG-RAST or DIAMOND-derived abundance summary counts
  (organism or functional category files) from ≥2 sample groups and need normalized
  differential transcript-activity calls with corrected p-values.
when_to_use_negative:
- The data are shotgun metagenomic (DNA) rather than metatranscriptomic counts — mRNA-based
  analysis measures transcriptional activity, not population size shifts, which it
  cannot measure as accurately as metagenomics.
- The input is already a normalized or pre-tested result table rather than raw per-sample
  abundance summary counts.
- Only one sample per condition is available — the DESeq2 negative-binomial group
  comparison requires replicate counts (e.g. 9 vs 6 samples).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R_organism_script.Rmd
  role: Imports organism summary files and performs the DESeq2 statistical significance
    testing on organism-level counts
  repo: https://github.com/transcript/SAMSA
- name: R_functional_script_w-md5s.Rmd
  role: Imports functional category summary files and performs statistical significance
    correlations on functional counts
  repo: https://github.com/transcript/SAMSA
- name: DESeq2
  role: Normalizes read counts and fits the negative-binomial model for the group
    contrast; DESeq statistics with adjusted p-values and log2FoldChange
- name: long_tail_threshold.py
  role: Optional pre-test filtering of rare taxa by thresholding summary files at
    a chosen cutoff percentage to remove low-abundance noise
  repo: https://github.com/transcript/SAMSA
- name: R
  role: Execution environment (preferably RStudio) for the .Rmd significance scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:393a0dcdbfb0db772be6ca802466bc18b92db3d88b7ae0b027ff689cc949ceb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - DESeq2
  - R
  - R_functional_script_w-md5s.Rmd
  - R_organism_script.Rmd
  - SAMSA
  - long_tail_threshold.py
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Count data normalization and multiple testing correction

## Summary

Statistical testing of metatranscriptome abundance count tables (organism or functional summary files) by normalizing raw counts with DESeq2's negative-binomial model and extracting features significant after multiple-testing correction, e.g. the Tyk2−/− vs wild-type DSS colitis comparison (Enterobacteriaceae 1.558-fold, p = 0.0055).

## When to use

Use when you have raw MG-RAST or DIAMOND-derived abundance summary counts (organism or functional category files) from ≥2 sample groups and need normalized differential transcript-activity calls with corrected p-values.

## When NOT to use

- The data are shotgun metagenomic (DNA) rather than metatranscriptomic counts — mRNA-based analysis measures transcriptional activity, not population size shifts, which it cannot measure as accurately as metagenomics.
- The input is already a normalized or pre-tested result table rather than raw per-sample abundance summary counts.
- Only one sample per condition is available — the DESeq2 negative-binomial group comparison requires replicate counts (e.g. 9 vs 6 samples).

## Inputs

- Organism abundance summary files from MG-RAST annotation results (sorted count tables from analysis_counter.py)
- Functional category summary files (e.g. from analysis_counter_w-md5s.py, retaining m5nr IDs)
- Sample metadata grouping (e.g. 9 Tyk2−/− vs 6 wild-type DSS colitis samples)

## Outputs

- Tabular significance results (per-feature p-value, adjusted p-value, log2FoldChange) at organism and functional levels
- Differentially expressed feature list at adjusted p < 0.05 (e.g. Enterobacteriaceae 1.558-fold, p = 0.0055; Butyrivibrio p = 0.03)
- Optional graphical representations of organism/functional activity

## How to apply

Import the aggregated organism summary files with R_organism_script.Rmd and functional summary files with R_functional_script_w-md5s.Rmd (R, preferably RStudio). Optionally first threshold rare, long-tail taxa with long_tail_threshold.py at a chosen cutoff percentage to remove low-abundance noise before testing. Let DESeq2 estimate size factors (normalizing for library depth) and fit the negative-binomial model across the group contrast — in the exemplar study 9 Tyk2−/− samples vs 6 wild-type controls. Extract features reaching adjusted p < 0.05 after multiple-testing correction, and report both p-value and log2FoldChange per feature in tabular output files. Verify biological expectations against known markers (e.g. increased Enterobacteriaceae, decreased Butyrivibrio, >300 differentially expressed protein-coding transcripts).

## Related tools

- **R_organism_script.Rmd** (Imports organism summary files and performs the DESeq2 statistical significance testing on organism-level counts) — https://github.com/transcript/SAMSA
- **R_functional_script_w-md5s.Rmd** (Imports functional category summary files and performs statistical significance correlations on functional counts) — https://github.com/transcript/SAMSA
- **DESeq2** (Normalizes read counts and fits the negative-binomial model for the group contrast; DESeq statistics with adjusted p-values and log2FoldChange)
- **long_tail_threshold.py** (Optional pre-test filtering of rare taxa by thresholding summary files at a chosen cutoff percentage to remove low-abundance noise) — https://github.com/transcript/SAMSA
- **R** (Execution environment (preferably RStudio) for the .Rmd significance scripts)

## Examples

```
Rscript run_DESeq_stats.R
```

## Evaluation signals

- All reported features carry both an adjusted p-value and a log2FoldChange; significance is defined as adjusted p < 0.05 after multiple-testing correction.
- Reproducing the exemplar analysis should recover a 1.558-fold Enterobacteriaceae increase (p = 0.0055) in day-3 Tyk2−/− vs wild-type samples, increased Escherichia and Providencia, and decreased Butyrivibrio (p = 0.03).
- More than 300 differentially expressed protein-coding transcripts should be detected at the transcript level in the exemplar dataset.
- Tabular output files must be produced for both organism-level and functional-level results.
- If long_tail_threshold.py is applied, low-abundance rare taxa below the chosen cutoff percentage should be absent from the tested summary files.

## Limitations

- Rare, long-tail taxa contribute noise to testing; thresholding at an arbitrary cutoff percentage can affect which features reach significance.
- Results depend on annotation depth: each sample needs 40–50 million raw sequences to yield the 5–10 million annotated reads necessary for accurate abundance measures.
- The comparison measures transcriptional activity, not microbial population size shifts, which cannot be measured as accurately as with shotgun metagenomics.

## Evidence

- [readme] R_organism_script.Rmd imports organism summary files and performs statistical significance testing: "R_organism_script.Rmd - imports organism summary files and performs statistical  significance correlations"
- [readme] R scripts generate tabular output with p-values and log2FoldChange per feature: [evidence span withheld — anchor status: external_doc]
- [readme] long_tail_threshold.py thresholds summary files at a chosen cutoff percentage: "long_tail_threshold.py - permits thresholding of summary files at a chosen cutoff percentage"
- [abstract] Significant differences between control and experimental groups in organism transcriptional activity and specific cellular functions: "We were also able to detect significant differences between control and experimental groups in both organism transcriptional activity and specific cellular functions."
- [abstract] Sufficient sequencing depth needed for accurate abundance measures: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
