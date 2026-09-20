---
name: seed-subsystems-hierarchy-parsing
description: Use when raw DIAMOND annotation output is line-by-line per-read text
  and you need condensed, descending-sorted organism or SEED Subsystems functional
  abundance tables for downstream statistics.
when_to_use_negative:
- Input is already a condensed feature/abundance table ready for DESeq import — aggregation
  would double-count or re-sort an already summarized file.
- Annotations come from a web service such as MG-RAST rather than local DIAMOND output;
  this skill operates on SAMSA2's per-read DIAMOND outfiles.
- Only per-read alignment statistics (e.g. identity, bit scores) are needed — this
  skill reduces lines to counts and discards alignment detail.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: SAMSA2
  role: Provides the Python aggregation scripts (DIAMOND_analysis_counter.py, DIAMOND_subsystems_analysis_counter.py,
    DIAMOND_specific_organism_counter.py) that condense per-read DIAMOND results into
    summary tables
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Produces the per-read annotation outfiles (BLAST-like, accelerated alignment
    against local databases) that are the input to aggregation
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:f145228b5a55a801bdace058b092751af47fd11cc32dc0e08e8d610ac6fbfdb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  - SEED Subsystems
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# seed-subsystems-hierarchy-parsing

## Summary

Aggregation-stage parsing of per-read DIAMOND annotation output into ranked abundance summary tables, preserving the SEED Subsystems hierarchical ontology levels (1/2/3) for functional annotations alongside organism-level counts.

## When to use

Use when raw DIAMOND annotation output is line-by-line per-read text and you need condensed, descending-sorted organism or SEED Subsystems functional abundance tables for downstream statistics.

## When NOT to use

- Input is already a condensed feature/abundance table ready for DESeq import — aggregation would double-count or re-sort an already summarized file.
- Annotations come from a web service such as MG-RAST rather than local DIAMOND output; this skill operates on SAMSA2's per-read DIAMOND outfiles.
- Only per-read alignment statistics (e.g. identity, bit scores) are needed — this skill reduces lines to counts and discards alignment detail.

## Inputs

- Per-read DIAMOND annotation outfile (one annotation line per read) from alignment against a local DIAMOND-indexed database
- User-supplied organism or function keyword (for the filtered subset mode)

## Outputs

- Ranked organism abundance summary table (feature label plus count, sorted descending)
- Ranked SEED Subsystems functional abundance summary preserving hierarchy levels 1/2/3 (feature label plus count, sorted descending)
- Keyword-filtered subset abundance table

## How to apply

After DIAMOND annotates reads against a local, DIAMOND-indexed database (e.g. RefSeq for organisms/specific functions, SEED Subsystems for hierarchical functional ontology), feed each per-read outfile to a Python aggregation script such as SAMSA2's DIAMOND_analysis_counter.py. The script counts occurrences of every annotation line, preserving SEED Subsystems levels 1/2/3 in the function table, then sorts both organism and function count tables in descending order of abundance to produce ranked abundance summaries. For targeted extraction, a keyword-filter variant (mirroring DIAMOND_specific_organism_counter.py) restricts the output to lines matching a user-supplied organism or function keyword. Because these summaries are one row per annotated feature plus its count, they import directly into R for run_DESeq_stats.R differential testing between control and experimental groups. SAMSA2 requires Python 2 for the aggregation scripts.

## Related tools

- **SAMSA2** (Provides the Python aggregation scripts (DIAMOND_analysis_counter.py, DIAMOND_subsystems_analysis_counter.py, DIAMOND_specific_organism_counter.py) that condense per-read DIAMOND results into summary tables) — https://github.com/transcript/samsa2
- **DIAMOND** (Produces the per-read annotation outfiles (BLAST-like, accelerated alignment against local databases) that are the input to aggregation) — https://github.com/bbuchfink/diamond

## Examples

```
python2 DIAMOND_analysis_counter.py annotated_reads.sam-converted.txt > organism_abundance.tsv
```

## Evaluation signals

- Reimplemented script runs without error on a real DIAMOND annotation outfile.
- Output matches the reference ranked abundance summary format: one feature label plus count per row, sorted in descending order of abundance.
- Organism table counts and functional table counts cover exactly the same set of annotated reads (two views of the same outfile).
- SEED Subsystems hierarchy levels 1/2/3 are retained as structured fields in the function table rather than flattened to a single label.
- Keyword-filter mode returns a subset table whose counts are <= the unfiltered counts and whose rows all match the supplied organism or function keyword.

## Limitations

- Aggregation scripts require Python 2; SAMSA2 works mostly with Python 3 but errors may not yet have been caught, and the pipeline was only tested on Linux.
- DIAMOND defaults to --fast mode, prioritizing speed over accuracy; sensitive mode raises memory requirements and changes the annotations being aggregated.
- Counts are per-read hits against the chosen local database — accuracy is bounded by that database (e.g. genus-level accuracy ~95% vs 91% for SAMSA 1.0 with MG-RAST).

## Evidence

- [methods] ranked abundance summary creation: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [discussion] SEED Subsystems for hierarchical ontologies: "and SEED Subsystems for hierarchical ontologies of functional activity groups"
- [readme] aggregation condenses line-by-line DIAMOND results: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt.  In the aggregation step, Python scripts condense these line-by-line results to create summary tables."
- [readme] SAMSA2 requires Python2 for aggregation scripts: [evidence span withheld — anchor status: external_doc]
- [readme] DESeq downstream of abundance summaries: [evidence span withheld — anchor status: not_found]
