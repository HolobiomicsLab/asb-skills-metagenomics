---
name: ranked-abundance-summary-aggregation
description: Use when you have raw per-read DIAMOND annotation output from SAMSA2
  (metatranscriptome reads annotated against local DIAMOND-indexed RefSeq or SEED
  Subsystems databases) and need ranked feature-abundance tables per sample.
when_to_use_negative:
- Input is already an aggregated feature/abundance table (e.g., a summary matrix ready
  for run_DESeq_stats.R) — re-aggregation is redundant
- The goal is de novo functional or taxonomic inference rather than annotation-based
  read counting against RefSeq/SEED references
- You only have MG-RAST-style pre-summarized annotations with no per-read DIAMOND
  output files to condense
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Provides the aggregation scripts (DIAMOND_analysis_counter.py) that condense
    per-read DIAMOND results into ranked abundance summaries; aggregation scripts
    require Python 2
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner producing the per-read annotation results (against
    local DIAMOND-indexed RefSeq/SEED databases) that feed the aggregation step; version
    0.8.3 used in the paper
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:3664c3171c92322294fab7bdba8b709a99b2f7b79a409ed8816ef21cd6fa20f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ranked-abundance-summary-aggregation

## Summary

Aggregates per-read DIAMOND annotation output from SAMSA2 metatranscriptome runs into ranked abundance summary tables of organism and functional features, using SAMSA2's Python aggregation scripts (e.g., DIAMOND_analysis_counter.py). This condenses line-item DIAMOND results into the tabular summaries needed for downstream DESeq statistics.

## When to use

Use when you have raw per-read DIAMOND annotation output from SAMSA2 (metatranscriptome reads annotated against local DIAMOND-indexed RefSeq or SEED Subsystems databases) and need ranked feature-abundance tables per sample.

## When NOT to use

- Input is already an aggregated feature/abundance table (e.g., a summary matrix ready for run_DESeq_stats.R) — re-aggregation is redundant
- The goal is de novo functional or taxonomic inference rather than annotation-based read counting against RefSeq/SEED references
- You only have MG-RAST-style pre-summarized annotations with no per-read DIAMOND output files to condense

## Inputs

- Per-read DIAMOND annotation output (line-item results) from metatranscriptome files
- Local DIAMOND-indexed reference database (NCBI RefSeq for organisms, SEED Subsystems for functions)
- Metatranscriptome FASTQ reads preprocessed by PEAR/Trimmomatic/SortMeRNA

## Outputs

- Ranked abundance summary table of organism or functional features per metatranscriptome file
- Tabular summaries importable into R for DESeq differential abundance statistics

## How to apply

Run DIAMOND (version 0.8.3 per the paper) to annotate each metatranscriptome file against a local, DIAMOND-indexed database — RefSeq for organisms or SEED Subsystems for hierarchical functional ontology — either via master_script.bash or example_DIAMOND_annotation_script.bash. DIAMOND returns per-read, line-item results, so run the aggregation step with SAMSA2's Python scripts: DIAMOND_analysis_counter.py condenses these line-by-line results into a ranked abundance summary for each metatranscriptome file. Note that SAMSA2 requires Python 2 for the aggregation scripts, so invoke them under Python 2.7 (SAMSA2 was tested on Linux). The resulting abundance summaries can then be imported into R and fed to run_DESeq_stats.R to determine significantly differing features between control vs. experimental groups. When annotation accuracy matters more than speed, switch DIAMOND from --fast (default) to --sensitive mode, accepting higher memory requirements.

## Related tools

- **SAMSA2** (Provides the aggregation scripts (DIAMOND_analysis_counter.py) that condense per-read DIAMOND results into ranked abundance summaries; aggregation scripts require Python 2) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner producing the per-read annotation results (against local DIAMOND-indexed RefSeq/SEED databases) that feed the aggregation step; version 0.8.3 used in the paper) — https://github.com/bbuchfink/diamond

## Examples

```
python2 DIAMOND_analysis_counter.py <diamond_results_file>  # create a ranked abundance summary of the DIAMOND results from each metatranscriptome file
```

## Evaluation signals

- Each output summary lists features ranked by abundance, with per-read DIAMOND line items fully condensed into summary tables (no unaggregated read lines remain)
- On MetaSim-simulated gut microbiome metatranscriptomes with known ground-truth labels, genus-level organism accuracy should approximate the reported ~95% (vs 91% for SAMSA 1.0)
- The aggregation script runs under Python 2.7 without error (SAMSA2 is tested on Linux; some Python 3 errors may not yet be caught)
- The ranked summaries import cleanly into R and run_DESeq_stats.R produces p-values and log2FoldChange scores per feature
- Annotation depends on the database being local and DIAMOND-indexed — a missing or stale index should fail loudly rather than yield empty summaries

## Limitations

- Aggregation scripts require Python 2.7; SAMSA2 works 'mostly' with Python 3 but some errors may not yet be caught (per README)
- DIAMOND is set by default to fast mode, which prioritizes speed over accuracy; --sensitive mode may significantly increase memory requirements
- Accuracy and coverage of summaries depend entirely on the local reference databases chosen (RefSeq/SEED Subsystems); custom databases must be indexed before use

## Evidence

- [methods] Ranked abundance summary creation: [evidence span withheld — anchor status: not_found]
- [readme] Aggregation step description: "In the aggregation step, Python scripts condense these line-by-line results to create summary tables."
- [readme] Python 2 requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmark accuracy: "Accuracy95% (genus level)91% (genus level)"
- [readme] DIAMOND mode trade-off: [evidence span withheld — anchor status: external_doc]
