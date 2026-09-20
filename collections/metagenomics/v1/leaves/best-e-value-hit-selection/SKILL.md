---
name: best-e-value-hit-selection
description: Use when processing per-sample DIAMAND tabular alignment output during
  metagenomic read mapping and you must reduce many-to-many read-gene hits to matches
  before gene counting.
when_to_use_negative:
- Hits come from a mapper whose scoring is not e-value-based, so minimum-e-value selection
  is meaningless
- You require strict single-best-hit assignment with no ties; this strategy deliberately
  keeps all equally-good tied best matches
- Input is not raw per-read alignment output but already an aggregated gene abundance/feature
  table
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaLAFFA
  role: Pipeline hosting the hit-filtering step (hit_filter.py / hit_filter_combine)
    implemented as a Snakemake workflow
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: DIAMOND
  role: Produces the tabular read-to-gene alignment hits that are filtered by best
    e-value
- name: Python
  role: Custom Python script performs the per-read best-e-value filtering and downstream
    fractional counting/normalization
- name: Snakemake
  role: Orchestrates the hit_filter and hit_filter_combine steps within the pipeline
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:702ad7474021c172117641482b8b53343b2411c5b3eff7a886fd3b51a17e82d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# best-e-value-hit-selection

## Summary

MetaLAFFA's default hit-refinement strategy for DIAMOND read-mapping output: per read, retain only the hit(s) with the best (minimum) e-value, keeping all equally-good tied best matches, which yields high-specificity functional profiling.

## When to use

Use when processing per-sample DIAMAND tabular alignment output during metagenomic read mapping and you must reduce many-to-many read-gene hits to best matches before gene counting.

## When NOT to use

- Hits come from a mapper whose scoring is not e-value-based, so minimum-e-value selection is meaningless
- You require strict single-best-hit assignment with no ties; this strategy deliberately keeps all equally-good tied best matches
- Input is not raw per-read alignment output but already an aggregated gene abundance/feature table

## Inputs

- Per-sample DIAMAND tabular alignment output (read-to-gene hits) from the map_reads_to_genes step
- Hit-filtering configuration in config/steps/hit_filter.py

## Outputs

- filtered_hits.tsv: tab-separated table of retained read-gene best-hit pairs, gzipped per input FASTQ
- Combined gzipped filtered-hit tables per sample (produced by hit_filter_combine)

## How to apply

Load the per-sample DIAMOND tabular alignment output produced by the map_reads_to_genes step of MetaLAFFA. Group all alignment hits by read identifier using the custom MetaLAFFA Python hit-filtering script (configured via config/steps/hit_filter.py in the Snakemake pipeline). For each read, scan its hits and keep only those with the best (minimum) e-value, retaining equally-good tied best matches and including genes that lack functional annotations, per the highest-specificity filtering strategy. Write the retained read-gene hit pairs to a tab-separated table (filtered_hits.tsv) and gzip it, since downstream hit_filter_combine concatenates gzipped per-FASTQ outputs for the same sample. Tied best hits are later handled by fractional gene counting, where a read matching equally well to multiple genes contributes a fractional count to each.

## Related tools

- **MetaLAFFA** (Pipeline hosting the hit-filtering step (hit_filter.py / hit_filter_combine) implemented as a Snakemake workflow) — https://github.com/borenstein-lab/MetaLAFFA
- **DIAMOND** (Produces the tabular read-to-gene alignment hits that are filtered by best e-value)
- **Python** (Custom Python script performs the per-read best-e-value filtering and downstream fractional counting/normalization)
- **Snakemake** (Orchestrates the hit_filter and hit_filter_combine steps within the pipeline) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
snakemake output/filtered_hits/<sample>.hits.tsv.gz
```

## Evaluation signals

- Every retained hit for a given read has the minimum e-value among that read's original alignment hits
- All equally-good tied best hits are retained per read (no arbitrary tie-breaking)
- Output is a gzipped tab-separated file (filtered_hits.tsv) per input FASTQ, with per-sample concatenation by hit_filter_combine
- Downstream gene counts show fractional contributions for reads that tie across multiple genes rather than full counts
- Unannotated genes are retained among best hits, consistent with the default strategy of allowing best hits to include genes without functional annotations

## Limitations

- Tied best matches distribute each read as fractional counts across multiple genes, which complicates strict per-gene read attribution
- MetaLAFFA's default installation pins Snakemake 3.13.3; the conda-based setup and UniProt database version are fixed defaults that may need adjustment
- The strategy trades sensitivity for specificity: hits that are not the best e-value are discarded even if biologically relevant

## Evidence

- [intro] Best match refinement of DIAMOND alignments: "MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database"
- [methods] best e-value hit filtering: "keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations."
- [methods] fractional gene counting: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [intro] Gene abundance calculation with fractional counts: "If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [readme] Hit filtering step in README: "5. Filtering read mapping hits"
