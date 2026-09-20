---
name: diamond-alignment-output-parsing
description: Use when you have DIAMOND tabular alignment hits of shotgun metagenomic
  reads against a microbial gene database (e.g. UniRef90) and must refine them to
  hits per read for gene abundance estimation.
when_to_use_negative:
- Input is already a gene count or gene abundance table rather than raw DIAMOND alignment
  hits
- Alignments are in SAM/BAM or other formats not produced by MetaLAFFA's DIAMOND tabular
  output
- A different hit-refinement strategy (not best-e-value per read) is required by the
  analysis design
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0114
tools:
- name: DIAMOND
  role: Produces the tabular read-to-gene alignment hits that are parsed and filtered
- name: MetaLAFFA
  role: Provides the Snakemake hit_filter step and custom Python hit-filtering script
    (config/steps/hit_filter.py) that performs best-match refinement
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom module-based scripts implement the hit filtering and fractional gene
    counting logic
- name: Snakemake
  role: Orchestrates the hit_filter and hit_filter_combine steps within the pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:23c5ecbc883dbd6187d8414686e1ba381d49cc76f7f7ab62203437d11218d558
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# diamond-alignment-output-parsing

## Summary

Parse DIAMOND tabular alignment output from MetaLAFFA's read-mapping phase and reduce it to best matches per read using the hit-refinement (best e-value hit filtering) strategy, yielding a gzipped read-gene hit table that feeds fractional gene counting.

## When to use

Use when you have DIAMOND tabular alignment hits of shotgun metagenomic reads against a microbial gene database (e.g. UniRef90) and must refine them to best hits per read for gene abundance estimation.

## When NOT to use

- Input is already a gene count or gene abundance table rather than raw DIAMOND alignment hits
- Alignments are in SAM/BAM or other formats not produced by MetaLAFFA's DIAMOND tabular output
- A different hit-refinement strategy (not best-e-value per read) is required by the analysis design

## Inputs

- Per-sample DIAMAND tabular alignment output of quality-controlled reads vs. microbial gene database (from MetaLAFFA map_reads_to_genes step)
- Hit-filtering configuration (config/steps/hit_filter.py within the Snakemake pipeline)

## Outputs

- Gzipped tab-separated read-gene hit table (filtered_hits.tsv.gz) containing only best-e-value hits per read
- Per-FASTQ outputs that are concatenated per sample by the hit_filter_combine step

## How to apply

Load the per-sample DIAMOND tabular alignment output produced by MetaLAFFA's map_reads_to_genes step, then group all hits by read identifier using MetaLAFFA's custom Python hit-filtering script configured under config/steps/hit_filter.py in the Snakemake pipeline. For each read, scan its hits and retain only those sharing the best (minimum) e-value, keeping all equally-good tied best matches — MetaLAFFA's highest-specificity default strategy — and including hits to genes that lack functional annotations. Write the retained read-gene pairs as a tab-separated table (filtered_hits.tsv) and gzip it, since the downstream hit_filter_combine step concatenates gzipped per-FASTQ outputs for the same sample. Downstream, if a read ties across multiple genes, each tied gene receives an even fractional count during gene counting, which the article notes is performed by a custom Python script together with gene length normalization.

## Related tools

- **DIAMOND** (Produces the tabular read-to-gene alignment hits that are parsed and filtered)
- **MetaLAFFA** (Provides the Snakemake hit_filter step and custom Python hit-filtering script (config/steps/hit_filter.py) that performs best-match refinement) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom module-based scripts implement the hit filtering and fractional gene counting logic)
- **Snakemake** (Orchestrates the hit_filter and hit_filter_combine steps within the pipeline)

## Evaluation signals

- Every read with at least one alignment retains one or more hits whose e-value equals the minimum e-value observed for that read
- Equally-good tied best hits are all retained (no arbitrary single-hit tie-breaking)
- Hits to genes lacking functional annotations are preserved in the filtered output
- Output is a gzipped TSV named filtered_hits.tsv per input FASTQ, and hit_filter_combine successfully concatenates all per-FASTQ outputs for a sample
- Per multi-match read, downstream fractional gene counts across tied genes sum to 1

## Limitations

- The default strategy optimizes specificity in functional profiling, which may reduce sensitivity for reads with many suboptimal hits
- Filtering is per-FASTQ and outputs must be combined (hit_filter_combine) before per-sample gene counting
- Fractional counting for multi-match reads means individual gene abundances are fractional rather than integer read counts

## Evidence

- [intro] Best match refinement of DIAMOND alignments: "MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database"
- [intro] Gene abundance calculation with fractional counts for multi-matches: "If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [methods] best e-value hit filtering: "keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations."
- [methods] fractional gene counting: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [readme] hit filter configuration: "config/steps/map_reads_to_genes.py, changing the number of cores defined in cluster_params to be the number of cores you want a single DIAMOND alignment process to use."
