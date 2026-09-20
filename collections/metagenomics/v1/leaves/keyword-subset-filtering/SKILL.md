---
name: keyword-subset-filtering
description: Use when you have a per-read DIAMOND annotation outfile from SAMSA2 and
  need a subset abundance table restricted to lines matching a specific organism or
  functional keyword.
when_to_use_negative:
- You already have a full ranked abundance summary and only need global feature counts
  rather than a keyword subset
- Input is raw FASTQ reads or the unaligned DIAMOND database output — annotation aggregation
  has not yet produced per-read annotation lines
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: SAMSA2
  role: Pipeline providing the Python aggregation/keyword-filter scripts (DIAMOND_specific_organism_counter.py,
    DIAMOND_analysis_counter.py) that produce subset abundance tables
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Produces the per-read annotation output (version 0.8.3) that the keyword filtering
    scripts consume
  repo: https://github.com/bbuchfink/diamond
- name: SEED Subsystems
  role: Hierarchical functional ontology whose level 1/2/3 labels are preserved in
    the filtered function tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:1a2a09fc6a21572cf0b52fbac708fc71d946cef5c8bd0daa8f0cf30b37e854c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  - SEED Subsystems
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Keyword subset filtering of DIAMOND annotation counts

## Summary

A SAMSA2 aggregation skill that filters per-read DIAMOND annotation lines by a user-supplied organism or function keyword and emits a ranked abundance subset table, mirroring DIAMOND_specific_organism_counter.py.

## When to use

Use when you have a per-read DIAMOND annotation outfile from SAMSA2 and need a subset abundance table restricted to lines matching a specific organism or functional keyword.

## When NOT to use

- You already have a full ranked abundance summary and only need global feature counts rather than a keyword subset
- Input is raw FASTQ reads or the unaligned DIAMOND database output — annotation aggregation has not yet produced per-read annotation lines

## Inputs

- Per-read DIAMOND annotation outfile from a metatranscriptome sample (line-item per-read annotation format)
- User-supplied organism or function keyword string

## Outputs

- Keyword-filtered ranked abundance table (feature label plus count, sorted descending), subset of the full DIAMOND abundance summary

## How to apply

Run the SAMSA2 aggregation scripts (Python 2.7, e.g. DIAMOND_specific_organism_counter.py) on the line-by-line DIAMOND output for a metatranscriptome sample. The script counts occurrences of every annotation line — preserving SEED Subsystems hierarchical levels (1/2/3) for functional tables — then filters to only the lines containing the user-supplied organism or function keyword. Sort the filtered counts in descending order of abundance and write the subset table with one row per annotated feature and its count. Rationale: DIAMOND returns per-read 'ticker tape' output that is too large to interpret directly; keyword filtering condenses it to just the features of interest before downstream DESeq statistics.

## Related tools

- **SAMSA2** (Pipeline providing the Python aggregation/keyword-filter scripts (DIAMOND_specific_organism_counter.py, DIAMOND_analysis_counter.py) that produce subset abundance tables) — https://github.com/transcript/samsa2
- **DIAMOND** (Produces the per-read annotation output (version 0.8.3) that the keyword filtering scripts consume) — https://github.com/bbuchfink/diamond
- **SEED Subsystems** (Hierarchical functional ontology whose level 1/2/3 labels are preserved in the filtered function tables)

## Examples

```
python2 DIAMOND_specific_organism_counter.py --infile sample1_DIAMOND_annotation.txt --search_term "Bacteroides" --outfile organism_subset_counts.txt
```

## Evaluation signals

- The filtered output contains only feature labels containing the supplied keyword — no unmatching lines present
- Counts in the subset table equal the corresponding counts in the full ranked abundance summary for matching features
- Rows are sorted in descending order of abundance, matching the reference DIAMOND_analysis_counter.py output format (feature label plus count)
- Functional tables preserve the SEED Subsystems level 1/2/3 hierarchy in the feature labels
- The script runs without error on a DIAMOND annotation outfile (Python 2.7)

## Limitations

- SAMSA2 aggregation scripts require Python 2.7; Python 3 support is incomplete ('there may be some errors not yet caught')
- Pipeline only tested on Linux despite claiming any OS supporting Python 2.7
- DIAMOND is set to --fast mode by default, prioritizing speed over accuracy, which affects the annotations being filtered

## Evidence

- [readme] Aggregation: DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt. In the aggregation step, Python scripts condense these line-by-line results to create summary tables.: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt. In the aggregation step, Python scripts condense these line-by-line results"
- [other] Keyword-filter mode emitting a subset table restricted to matching lines: [evidence span withheld — anchor status: not_found]
- [methods] Ranked abundance summary creation in methods: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [discussion] SEED Subsystems used for hierarchical functional ontology: "and SEED Subsystems for hierarchical ontologies of functional activity groups"
- [readme] Python 2 dependency for aggregation scripts: [evidence span withheld — anchor status: external_doc]
