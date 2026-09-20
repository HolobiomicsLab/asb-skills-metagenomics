---
name: annotation-abundance-counting
description: Use when you have raw per-read annotation output files from MG-RAST (or
  DIAMOND) and need to condense them into abundance summary tables of organism or
  functional hits before statistical comparison.
when_to_use_negative:
- Annotation has not yet been performed - files must first be submitted to and returned
  by the MG-RAST server (or annotated locally with DIAMOND in SAMSA2); counting cannot
  precede annotation
- Input is already a condensed summary table (e.g. output of RefSeq_output_reducer.py
  or func_data_trimmer.py); re-counting would only duplicate existing tallies
- You need p-values or log2FoldChange scores for differential expression - that is
  the separate R/DESeq2 analysis step (R_organism_script.Rmd, R_functional_script_w-md5s.Rmd),
  not the aggregation counting step
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: analysis_counter.py
  role: 'Primary aggregation script: creates a sorted summary of abundance counts
    from MG-RAST annotation results files by tallying unique annotations and exporting
    them'
  repo: https://github.com/transcript/SAMSA
- name: analysis_counter_w-md5s.py
  role: Variant of the counter that performs the same aggregation but maintains the
    m5nr ID from MG-RAST for later transcript identification
  repo: https://github.com/transcript/SAMSA
- name: DIAMOND_analysis_counter.py
  role: SAMSA2 equivalent that creates a ranked abundance summary from DIAMOND per-read
    annotation results instead of MG-RAST output
  repo: https://github.com/transcript/samsa2
- name: MG-RAST
  role: Annotation engine whose per-read output files are the direct input to the
    counting step (version 3.6 used)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:801cb335e0c2cb86b72aca8572a59b2d0f2b6637e203e0b067bed4bfb932cdaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - DIAMOND_analysis_counter.py
  - MG-RAST
  - SAMSA
  - analysis_counter.py
  - analysis_counter_w-md5s.py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# annotation-abundance-counting

## Summary

Aggregation-phase skill from the SAMSA pipeline: parse per-read MG-RAST annotation results line by line, tally occurrences of each unique annotation in a dictionary, and export a sorted, tab-delimited abundance summary for downstream DESeq2 analysis.

## When to use

Use when you have raw per-read annotation output files from MG-RAST (or DIAMOND) and need to condense them into abundance summary tables of organism or functional hits before statistical comparison.

## When NOT to use

- Annotation has not yet been performed - files must first be submitted to and returned by the MG-RAST server (or annotated locally with DIAMOND in SAMSA2); counting cannot precede annotation
- Input is already a condensed summary table (e.g. output of RefSeq_output_reducer.py or func_data_trimmer.py); re-counting would only duplicate existing tallies
- You need p-values or log2FoldChange scores for differential expression - that is the separate R/DESeq2 analysis step (R_organism_script.Rmd, R_functional_script_w-md5s.Rmd), not the aggregation counting step

## Inputs

- Tab-delimited per-read annotation results file downloaded from the MG-RAST server (one annotation hit per read)
- Optionally: DIAMOND per-read annotation output (SAMSA2's DIAMOND_analysis_counter.py accepts the same kind of ranked-input)
- Sample file name/label so the summary can be grouped for later control-vs-experimental comparison in R

## Outputs

- Tab-delimited abundance summary file (e.g. annotation_summary.tsv) with one row per unique annotation, sorted by non-increasing abundance count
- Optionally: summary file retaining MG-RAST m5nr IDs per annotation (analysis_counter_w-md5s.py)

## How to apply

Load the tab-delimited MG-RAST annotation results file with Python 2.7 and parse each annotation record line by line. Tally occurrences of each unique annotation in a dictionary, incrementing a counter per observed hit. Sort the annotation-count dictionary in descending order of abundance and export it as a tab-delimited abundance summary (e.g. annotation_summary.tsv) formatted so the SAMSA R scripts (R_organism_script.Rmd, R_functional_script_w-md5s.Rmd) can import it for DESeq2 statistical testing. If transcript identification against m5nr IDs is required downstream, use analysis_counter_w-md5s.py instead, which preserves the MG-RAST m5nr ID per annotation.

## Related tools

- **analysis_counter.py** (Primary aggregation script: creates a sorted summary of abundance counts from MG-RAST annotation results files by tallying unique annotations and exporting them) — https://github.com/transcript/SAMSA
- **analysis_counter_w-md5s.py** (Variant of the counter that performs the same aggregation but maintains the m5nr ID from MG-RAST for later transcript identification) — https://github.com/transcript/SAMSA
- **DIAMOND_analysis_counter.py** (SAMSA2 equivalent that creates a ranked abundance summary from DIAMOND per-read annotation results instead of MG-RAST output) — https://github.com/transcript/samsa2
- **MG-RAST** (Annotation engine whose per-read output files are the direct input to the counting step (version 3.6 used))

## Examples

```
python analysis_counter.py -usage
```

## Evaluation signals

- The summary file exists after the run and is tab-delimited and importable by the SAMSA R scripts
- Row count in the summary equals the number of unique annotations tallied from the input annotation file (one row per unique annotation)
- Rows are ordered by non-increasing abundance (the file is a ranked abundance summary)
- Every count in the summary is the sum of occurrences of that annotation observed in the per-read input - no annotation hits are dropped or double-counted
- In SAMSA2 mode, DIAMOND_analysis_counter.py output is directly accepted as input by run_DESeq_stats.R without reformatting

## Limitations

- Requires Python 2.7 specifically for the aggregation scripts (SAMSA2 notes Python3 'may have some errors not yet caught')
- Accurate abundance measures require sufficient depth: ~40-50 million raw sequences per sample, expected to yield 5-10 million annotated reads
- MG-RAST annotation servers require queue waits of days to weeks, so obtaining the input files for counting can be slow
- Remaining ribosomal reads must be discarded before counting/aggregation, as rRNA hits do not reflect the mRNA profile
- For eukaryotic organisms, MG-RAST's elevated mismatch rate means the annotation tallies feeding this step may be less reliable

## Evidence

- [readme] analysis_counter.py - creates a sorted summary of abundance counts from MG-RAST annotation results files: [evidence span withheld — anchor status: external_doc]
- [readme] After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2.: "After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2."
- [readme] In the aggregation step, Python scripts condense these line-by-line results to create summary tables.: [evidence span withheld — anchor status: external_doc]
- [discussion] Our pipeline fulfills the need for four main steps in metatranscriptomic data analysis: preprocessing, annotation, aggregation, and analysis.: "Our pipeline fulfills the need for four main steps in metatranscriptomic data analysis: preprocessing, annotation, aggregation, and analysis."
- [abstract] Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures.: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
