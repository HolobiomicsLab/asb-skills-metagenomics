---
name: sorted-ranked-table-generation
description: Use when you have per-read DIAMOND annotation output from a metatranscriptome
  sample and need to convert it into a sorted, ranked abundance summary table of organisms
  or functional annotations.
when_to_use_negative:
- Input is already a summary/abundance feature table — this skill consumes raw per-read
  DIAMOND line-item output, not pre-aggregated tables
- Data has not yet been through preprocessing and annotation — the pipeline order
  is preprocessing, annotation, then aggregation
- The analysis requires raw alignment statistics rather than abundance counts (e.g.,
  assessing alignment quality itself)
tools:
- name: SAMSA2
  role: Pipeline whose aggregation scripts (DIAMOND_analysis_counter.py, DIAMOND_subsystems_analysis_counter.py,
    DIAMOND_specific_organism_counter.py) condense DIAMOND per-read results into sorted
    abundance summaries
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner whose per-read annotation output is the input
    to the aggregation step
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:475430f6bdf8bcf0c38d36866d4a2fa352eb570b1b03a258741377ec8df5bf11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sorted-ranked-table-generation

## Summary

Reimplements the SAMSA2 aggregation step: Python scripts condense per-read DIAMOND annotation output into sorted abundance count tables, producing ranked summaries of organism annotations and SEED Subsystems functional annotations. This converts line-item annotation receipts into summary files usable for DESeq differential analysis.

## When to use

Use when you have per-read DIAMOND annotation output from a metatranscriptome sample and need to convert it into a sorted, ranked abundance summary table of organisms or functional annotations.

## When NOT to use

- Input is already a summary/abundance feature table — this skill consumes raw per-read DIAMOND line-item output, not pre-aggregated tables
- Data has not yet been through preprocessing and annotation — the pipeline order is preprocessing, annotation, then aggregation
- The analysis requires raw alignment statistics rather than abundance counts (e.g., assessing alignment quality itself)

## Inputs

- Per-read DIAMOND annotation outfile (one annotation line per read) from a metatranscriptome sample
- Optional user-supplied organism or function keyword for the filtered-subset mode

## Outputs

- Ranked abundance summary TSV (feature label plus count, sorted descending), one row per annotated feature
- Organism annotation count table and SEED Subsystems functional annotation count table for the same reads
- Optional keyword-filtered subset table restricted to matching annotation lines

## How to apply

Read each per-read annotation outfile produced by DIAMOND alignment of the metatranscriptome sample. Count occurrences of every organism and functional annotation line, preserving the SEED Subsystems hierarchical levels (1/2/3) in the function table, mirroring DIAMOND_analysis_counter.py and DIAMOND_subsystems_analysis_counter.py. Sort both count tables in descending order of abundance to create a ranked abundance summary of the DIAMOND results. Optionally add a keyword-filter mode, as in DIAMOND_specific_organism_counter.py, that emits a subset table restricted to lines matching a user-supplied organism or function keyword. The aggregation step is needed because DIAMOND returns results on a per-read basis, 'a bit like a ticker tape or a line item receipt', and Python scripts condense these line-by-line results into summary tables.

## Related tools

- **SAMSA2** (Pipeline whose aggregation scripts (DIAMOND_analysis_counter.py, DIAMOND_subsystems_analysis_counter.py, DIAMOND_specific_organism_counter.py) condense DIAMOND per-read results into sorted abundance summaries) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner whose per-read annotation output is the input to the aggregation step) — https://github.com/bbuchfink/diamond

## Examples

```
python2 DIAMOND_analysis_counter.py sample1_DIAMOND_annotation_outfile.txt
```

## Evaluation signals

- The script runs without error on a real DIAMOND annotation outfile
- Output matches the reference ranked abundance summary format produced by DIAMOND_analysis_counter.py: one row per annotated feature with its count
- Rows are sorted in descending order of abundance
- Both an organism table and a functional (SEED Subsystems) table are produced for the same set of reads
- Keyword-filter mode emits exactly the subset of lines matching the user-supplied keyword

## Limitations

- The aggregation scripts require Python 2.7; SAMSA2 was tested on Linux and the pipeline works mostly with Python 3, although there may be errors not yet caught
- Aggregation is step 3 of 4 and assumes the DIAMOND annotation step already ran against a local, DIAMOND-indexed database
- Counts are per-read annotation tallies; differential testing downstream (DESeq) is handled by separate R scripts, not by this step

## Evidence

- [readme] Aggregation condenses DIAMOND per-read results: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt. In the aggregation step, Python scripts condense these line-by-line results to create summary tables."
- [methods] Ranked abundance summary via counter script: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [methods] Pipeline ordering of aggregation: "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
- [readme] DESeq consumes the abundance summaries: [evidence span withheld — anchor status: external_doc]
- [readme] Python 2.7 dependency for aggregation: [evidence span withheld — anchor status: external_doc]
