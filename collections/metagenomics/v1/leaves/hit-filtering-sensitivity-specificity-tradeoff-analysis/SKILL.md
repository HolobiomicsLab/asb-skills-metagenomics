---
name: hit-filtering-sensitivity-specificity-tradeoff-analysis
description: Use when choosing or configuring a hit-filtering strategy for DIAMOND
  alignment output in read-based functional annotation, or when reasoning about sensitivity/specificity
  trade-offs from multi-match reads.
when_to_use_negative:
- Input is not a per-read alignment hit table but an already-aggregated gene or ortholog
  abundance table
- The alignment was produced by a tool whose output schema differs from DIAMOND tabular
  format without adaptation
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: MetaLAFFA
  role: Pipeline providing the hit-refinement step and its configurable filtering
    strategies via config/steps/hit_filter.py
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: DIAMOND
  role: Produces the tabular read-to-gene alignment hits that feed the hit-filtering
    step
- name: Snakemake
  role: Workflow engine in which the hit-filtering step (hit_filter and hit_filter_combine)
    is orchestrated
- name: Python
  role: Custom hit-filtering script implements the per-read best e-value strategy
    and fractional gene counting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:b57e8d1569e0329fb4af3809271259105cfb52de0df56245cc044b63888f0022
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# hit-filtering-sensitivity-specificity-tradeoff-analysis

## Summary

Analyzes how alternative read-mapping hit refinement strategies (best e-value match with tie retention vs. keeping all hits) affect sensitivity and specificity of metagenomic functional profiling, as implemented in MetaLAFFA between DIAMOND alignment and gene counting.

## When to use

Use when choosing or configuring a hit-filtering strategy for DIAMOND alignment output in read-based functional annotation, or when reasoning about sensitivity/specificity trade-offs from multi-match reads.

## When NOT to use

- Input is not a per-read alignment hit table but an already-aggregated gene or ortholog abundance table
- The alignment was produced by a tool whose output schema differs from DIAMOND tabular format without adaptation

## Inputs

- Per-sample DIAMAND tabular alignment output of read-to-gene hits (unfiltered hits from map_reads_to_genes)
- MetaLAFFA hit-filtering configuration (config/steps/hit_filter.py)

## Outputs

- Filtered read-gene hit table (filtered_hits.tsv), gzipped
- Concatenated per-sample filtered hits (via hit_filter_combine)
- Gene abundance counts with fractional counts for reads matching equally well to multiple genes

## How to apply

Load the per-sample DIAMOND tabular alignment output from MetaLAFFA's map_reads_to_genes step and group hits by read identifier. For each read, apply the chosen strategy: MetaLAFFA's default 'highest-specificity' strategy keeps only hits with the best (minimum) e-value, retaining all equally-good tied best matches and including genes without functional annotations; alternatives favor sensitivity by keeping all hits. When a read ties across multiple genes, the downstream gene-counting step assigns each tied gene an even fractional count rather than a full count. Configuration of the strategy lives in config/steps/hit_filter.py within the Snakemake pipeline, and outputs are written as a tab-separated table (filtered_hits.tsv) and gzipped; hit_filter_combine then concatenates the gzipped per-FASTQ outputs for the same sample. Judge correctness by verifying every retained hit has the minimum e-value among that read's hits, tie behavior is consistent, and the fractional-count convention is preserved downstream.

## Related tools

- **MetaLAFFA** (Pipeline providing the hit-refinement step and its configurable filtering strategies via config/steps/hit_filter.py) — https://github.com/borenstein-lab/MetaLAFFA
- **DIAMOND** (Produces the tabular read-to-gene alignment hits that feed the hit-filtering step)
- **Snakemake** (Workflow engine in which the hit-filtering step (hit_filter and hit_filter_combine) is orchestrated)
- **Python** (Custom hit-filtering script implements the per-read best e-value strategy and fractional gene counting)

## Evaluation signals

- Every retained hit for a read has the minimum e-value among that read's candidate hits (no suboptimal e-value hits survive)
- Equally-good tied best matches are all retained, and tied genes each receive an even fractional count in gene abundances
- Output is a tab-separated filtered_hits table, gzipped, as expected by hit_filter_combine
- Hits to genes lacking functional annotations are retained under the default highest-specificity strategy
- Summary statistics for the filtering step show plausible retained-hit fractions relative to the 53%-78% read-matching rates reported for UniRef90

## Limitations

- The default best-e-value strategy trades sensitivity for specificity: reads with multiple distinct plausible gene matches contribute fractional rather than full counts
- Reads mapping to unannotated genes are kept by default, which can retain hits that cannot be translated to functional orthology groups
- The strategy is defined in config/steps/hit_filter.py and must be reconfigured explicitly to change behavior

## Evidence

- [intro] Best match refinement of DIAMOND alignments: "MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database"
- [intro] Gene abundance calculation with fractional counts: "If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [methods] best e-value hit filtering: "keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations."
- [methods] fractional gene counting: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [readme] Filtering read mapping hits: "4. Mapping reads to gene IDs
5. Filtering read mapping hits
6. Gene counting
7. Ortholog counting
8. Correcting orthology group abundances"
