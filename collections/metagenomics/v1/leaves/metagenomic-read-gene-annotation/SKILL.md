---
name: metagenomic-read-gene-annotation
description: Use when per-sample DIAMOND tabular hits against a gene catalog (e.g.
  UniRef90) need -e-value refinement and fractional gene counting to produce gene/orthology
  abundances from shotgun metagenomic FASTQs.
when_to_use_negative:
- Input is already a filtered hit table or a gene count/abundance table — resume MetaLAFFA
  from a later step instead of re-filtering
- You need all alignment hits (sensitivity-first profiling, e.g. strain-level or variant
  analysis) rather than the specificity-oriented best-e-value-only strategy
- The data are assembled contigs or metatranscriptomes rather than quality-controlled
  shotgun metagenomic reads destined for DIAMOND-vs-UniRef90 mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: MetaLAFFA
  role: End-to-end Snakemake pipeline whose map_reads_to_genes, hit_filter, and hit_filter_combine
    steps implement DIAMOND mapping, best-e-value hit refinement, and per-sample merging
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: DIAMOND
  role: Aligner used to map reads to the UniRef90 gene database, producing the tabular
    hit file that the hit-filtering script refines
- name: Snakemake
  role: Workflow engine that runs the per-FASTQ hit filtering step configured in config/steps/hit_filter.py
    and supports distributed execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:ae0dd6a2bbfb31f7b7e1fb511f418ba8ac7ab48dc354932376550b7cb39470cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metagenomic-read-gene-annotation

## Summary

MetaLAFFA's read mapping phase aligns quality-controlled shotgun reads to a microbial gene database (UniRef90 by default) with DIAMOND, then refines the raw hits by keeping only the best-e-value match(es) per read and computes gene abundances with fractional counts for ties. This default best-match filtering strategy prioritizes specificity in functional profiling.

## When to use

Use when per-sample DIAMOND tabular hits against a gene catalog (e.g. UniRef90) need best-e-value refinement and fractional gene counting to produce gene/orthology abundances from shotgun metagenomic FASTQs.

## When NOT to use

- Input is already a filtered hit table or a gene count/abundance table — resume MetaLAFFA from a later step instead of re-filtering
- You need all alignment hits (sensitivity-first profiling, e.g. strain-level or variant analysis) rather than the specificity-oriented best-e-value-only strategy
- The data are assembled contigs or metatranscriptomes rather than quality-controlled shotgun metagenomic reads destined for DIAMOND-vs-UniRef90 mapping

## Inputs

- quality-controlled gzipped shotgun metagenomic FASTQ files (per sample/per FASTQ)
- prepared UniRef90 protein sequence database for DIAMOND
- DIAMOND tabular alignment output mapping reads to genes (per FASTQ)
- KEGG ko-to-module and ko-to-pathway mappings (for downstream orthology aggregation)

## Outputs

- filtered_hits.tsv.gz: gzipped per-FASTQ table of read-gene hits restricted to the best e-value (ties retained)
- per-sample combined hit table produced by hit_filter_combine
- gene abundance table with fractional counts for reads tied across genes and gene length normalization
- orthology group (KO) abundances aggregated into modules and pathways

## How to apply

In the MetaLAFFA Snakemake pipeline, run map_reads_to_genes to align gzipped per-FASTQ reads against the prepared UniRef90 protein database with DIAMOND, yielding a per-FASTQ tabular alignment file. Then execute the hit_filter step, configured in config/steps/hit_filter.py: the custom Python script groups all hits by read identifier and retains only hits with the best (minimum) e-value, keeping all equally-good tied best matches and including genes that lack functional annotations. Write the retained read-gene pairs to filtered_hits.tsv and gzip it, since the downstream hit_filter_combine step concatenates gzipped per-FASTQ outputs belonging to the same sample. During gene counting, each read tied equally across multiple genes contributes a fractional count to each tied gene, and counts are normalized by gene length. Judge correctness by checking that every read in the output carries only minimum-e-value hits, ties are preserved rather than arbitrarily broken, and per-FASTQ outputs were merged per sample before gene counting.

## Related tools

- **MetaLAFFA** (End-to-end Snakemake pipeline whose map_reads_to_genes, hit_filter, and hit_filter_combine steps implement DIAMOND mapping, best-e-value hit refinement, and per-sample merging) — https://github.com/borenstein-lab/MetaLAFFA
- **DIAMOND** (Aligner used to map reads to the UniRef90 gene database, producing the tabular hit file that the hit-filtering script refines)
- **Snakemake** (Workflow engine that runs the per-FASTQ hit filtering step configured in config/steps/hit_filter.py and supports distributed execution)

## Examples

```
prepare_databases.py -hr -km -u
```

## Evaluation signals

- For every read identifier in filtered_hits.tsv, all retained hits share the minimum e-value observed for that read across the raw DIAMOND output
- Equally-good tied best hits are all retained (a read may map to multiple genes), and genes without functional annotations are not dropped
- Per-FASTQ hit files are gzipped and hit_filter_combine successfully concatenates all files belonging to the same sample before counting
- Gene abundance table shows fractional (non-integer) counts for reads tied across multiple genes, after gene length normalization
- MetaLAFFA step summary statistics report plausible mapping rates (in the paper, 53%–78% of reads matched UniRef90 across HMP stool samples)

## Limitations

- The default best-match strategy optimizes specificity at the cost of sensitivity — sub-optimal hits are discarded, which loses information for divergent reads
- Fractional counting splits a read's contribution evenly among tied genes, diluting per-gene abundances when many ties occur
- Filtering uses e-value alone; bit score, identity, and alignment length are not tie-breakers
- MetaLAFFA's FAQ notes it defaults to an older version of UniProt/UniRef90, and database preparation is resource-intensive (several hours, ~40GB RAM, ~150–300GB disk)

## Evidence

- [intro] MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database: "MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database"
- [methods] keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations.: "keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations."
- [intro] gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count: "gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [results] The percentage of reads that had a match in the UniRef90 [21] database varied from 53% to 78% across samples: "The percentage of reads that had a match in the UniRef90 [21] database varied from 53% to 78% across samples"
