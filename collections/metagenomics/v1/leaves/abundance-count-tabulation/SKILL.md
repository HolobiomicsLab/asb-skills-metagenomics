---
name: abundance-count-tabulation
description: Use when DIAMOND alignment of metatranscriptome reads has produced per-read
  annotation outfiles and ranked organism/function abundance summaries are needed
  for downstream DESeq statistics.
when_to_use_negative:
- Input is already a summary/count table — the aggregation scripts expect raw per-read
  line-by-line DIAMOND output
- Reads have not yet passed preprocessing (PEAR merging, Trimmomatic cleaning, SortMeRNA
  ribosomal filtering) or DIAMOND annotation, so no per-read annotation file exists
- The task is differential abundance statistics itself — that is handled downstream
  by run_DESeq_stats.R on the already-created abundance summaries
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: SAMSA2
  role: Provides the Python aggregation scripts (DIAMOND_analysis_counter.py, DIAMOND_subsystems_analysis_counter.py,
    DIAMOND_specific_organism_counter.py) that condense per-read DIAMOND output into
    ranked abundance summary tables
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Upstream aligner (version 0.8.3 used in SAMSA2) that produces the per-read
    annotation outfile against a local, DIAMOND-indexed database, which is the input
    to the aggregation step
  repo: https://github.com/bbuchfink/diamond
- name: SEED Subsystems
  role: Reference database whose hierarchical functional ontology (level 1/2/3) is
    preserved in the functional abundance summary produced by the aggregation step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:7e6069600b2b080274987739769f6e6bfbc7532e35b013ed9b530ec4279cad54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  - SEED Subsystems
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# abundance-count-tabulation

## Summary

Aggregation-step skill from the SAMSA2 metatranscriptome pipeline: convert per-read DIAMOND annotation output (line-by-line results, 'like a ticker tape') into ranked abundance summary tables via Python counting scripts, producing separate organism and functional (SEED Subsystems) feature counts.

## When to use

Use when DIAMOND alignment of metatranscriptome reads has produced per-read annotation outfiles and ranked organism/function abundance summaries are needed for downstream DESeq statistics.

## When NOT to use

- Input is already a summary/count table — the aggregation scripts expect raw per-read line-by-line DIAMOND output
- Reads have not yet passed preprocessing (PEAR merging, Trimmomatic cleaning, SortMeRNA ribosomal filtering) or DIAMOND annotation, so no per-read annotation file exists
- The task is differential abundance statistics itself — that is handled downstream by run_DESeq_stats.R on the already-created abundance summaries

## Inputs

- Per-read DIAMOND annotation outfile from a metatranscriptome sample (one annotation per line)
- User-supplied organism or function keyword (optional, for the specific-counter filter mode)

## Outputs

- Ranked abundance summary table of organism annotations (feature label plus count, sorted descending)
- Ranked abundance summary table of functional annotations preserving SEED Subsystems level 1/2/3 hierarchy
- Keyword-restricted subset abundance table (optional)
- Tabular summaries importable into R for DESeq differential abundance analysis

## How to apply

Take each per-read DIAMOND annotation outfile and use Python aggregation scripts (in SAMSA2: DIAMOND_analysis_counter.py for organism annotations, DIAMOND_subsystems_analysis_counter.py for hierarchical SEED Subsystems function annotations) to count occurrences of every annotation line, preserving SEED level 1/2/3 hierarchy in the function table. Sort the resulting count tables in descending abundance to create a ranked abundance summary per metatranscriptome file. Optionally apply a keyword-filter mode (as in DIAMOND_specific_organism_counter.py) to emit a subset table restricted to a user-supplied organism or function keyword. Write the ranked counts to a summary file (e.g. abundance_summary.tsv) with one row per annotated feature and its count; these summaries are the direct input to R scripts like run_DESeq_stats.R. The aggregation scripts require Python 2 (SAMSA2 notes Python2 is required for aggregation scripts and the pipeline is 'tested on Linux' with Python 2.7).

## Related tools

- **SAMSA2** (Provides the Python aggregation scripts (DIAMOND_analysis_counter.py, DIAMOND_subsystems_analysis_counter.py, DIAMOND_specific_organism_counter.py) that condense per-read DIAMOND output into ranked abundance summary tables) — https://github.com/transcript/samsa2
- **DIAMOND** (Upstream aligner (version 0.8.3 used in SAMSA2) that produces the per-read annotation outfile against a local, DIAMOND-indexed database, which is the input to the aggregation step) — https://github.com/bbuchfink/diamond
- **SEED Subsystems** (Reference database whose hierarchical functional ontology (level 1/2/3) is preserved in the functional abundance summary produced by the aggregation step)

## Examples

```
python DIAMOND_analysis_counter.py <diamond_annotation_outfile>
```

## Evaluation signals

- The output is a ranked table: one row per annotated feature with its count, sorted in descending order of abundance
- Two matched summaries exist for the same reads: one for organism annotations and one for functional (SEED Subsystems) annotations
- Counts in the summary equal the number of annotation lines for that feature in the per-read DIAMOND outfile
- In the function table, SEED Subsystems hierarchical levels (1/2/3) are preserved in the feature labels
- The script runs without error on a real DIAMOND annotation outfile and its output matches the reference format produced by DIAMOND_analysis_counter.py

## Limitations

- SAMSA2 requires Python2 for the aggregation scripts; the pipeline works mostly with Python3 but 'there may be some errors not yet caught'
- Pipeline is stated for 'any supporting Python 2.7' but was only tested on Linux
- DIAMOND output is per-read only; the aggregation step cannot recover information absent from the annotation lines (e.g. it operates after SortMeRNA has already discarded ribosomal reads)

## Evidence

- [readme] Per-read output condensed into summary tables by Python aggregation scripts: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt. In the aggregation step, Python scripts condense these line-by-line results to create summary tables."
- [methods] Ranked abundance summary created by DIAMOND_analysis_counter.py: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [readme] Python2 requirement for aggregation scripts: [evidence span withheld — anchor status: external_doc]
- [methods] DESeq downstream analysis consumes the abundance summaries: "use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups"
- [methods] Aggregation is one of the four pipeline phases named in the methods: "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
