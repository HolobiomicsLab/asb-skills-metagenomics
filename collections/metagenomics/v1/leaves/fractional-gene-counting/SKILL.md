---
name: fractional-gene-counting
description: Use when building per-gene abundance tables from hit-filtered DIAMOND
  read-to-gene alignments ( e-value hits per read) and you need MetaLAFFA-compatible
  fractional counts for reads tied across multiple genes.
when_to_use_negative:
- Input is already an aggregated abundance or feature table rather than raw read-to-gene
  alignment hits — recounting would double-apply normalization.
- You need per-ortholog or per-pathway abundances rather than per-gene abundances;
  MetaLAFFA handles those in separate ortholog counting and aggregation steps.
- Your alignment hits have not been filtered to best e-value matches, so ties across
  equally-good genes are not yet defined.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaLAFFA
  role: Defines the fractional gene counting step and the gene length normalization
    applied to read-to-gene hits
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom Python script performs gene length normalization and fractional counting
    of multi-mapping reads
  repo: https://github.com/engal/MetaLAFFA
- name: DIAMOND
  role: Produces the read-to-gene alignments whose filtered hits are the input to
    fractional counting
- name: Snakemake
  role: Pipeline framework that orchestrates the gene counting step alongside the
    other MetaLAFFA steps
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:c0b609e532389896bb01a414da5294db41b3e88cebda3830bf4d43023a499107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# fractional-gene-counting

## Summary

Reconstructs MetaLAFFA's gene abundance calculation, in which per-gene abundances are obtained by summing retained DIAMOND read hits, and reads matching equally well to multiple genes contribute a fractional count (1/n) distributed evenly across those genes, followed by gene-length normalization.

## When to use

Use when building per-gene abundance tables from hit-filtered DIAMOND read-to-gene alignments (best e-value hits per read) and you need MetaLAFFA-compatible fractional counts for reads tied across multiple genes.

## When NOT to use

- Input is already an aggregated abundance or feature table rather than raw read-to-gene alignment hits — recounting would double-apply normalization.
- You need per-ortholog or per-pathway abundances rather than per-gene abundances; MetaLAFFA handles those in separate ortholog counting and aggregation steps.
- Your alignment hits have not been filtered to best e-value matches, so ties across equally-good genes are not yet defined.

## Inputs

- gzipped, hit-filtered DIAMOND alignment output (best e-value hits per read) for each sample
- gene length normalization table (.norm file) with each gene's length and normalization factor (gene length / average gene length)

## Outputs

- per-sample gene abundance counts (fractional, length-normalized)
- single tab-separated gene abundance table with genes as rows and samples as columns

## How to apply

Load the hit-filtered DIAMOND alignment output (gzipped best e-value hits per read) and the gene length normalization table (.norm file), where each gene's normalization factor is gene length divided by the average gene length. Sum retained hits per gene with a custom Python script implementing MetaLAFFA's fractional counting: each read that matches equally well to n genes contributes 1/n to each of those genes' abundances instead of a full count. Then divide each gene's fractional read count by its length normalization factor, correcting for longer genes being more likely to be sequenced. Finally, assemble the per-sample counts into a single tab-separated abundance table with genes (e.g. UniRef90) as rows and samples as columns. The fractional scheme is MetaLAFFA's default because strict best-hit selection can leave arbitrary ties that would otherwise be double-counted or arbitrarily assigned.

## Related tools

- **MetaLAFFA** (Defines the fractional gene counting step and the gene length normalization applied to read-to-gene hits) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom Python script performs gene length normalization and fractional counting of multi-mapping reads) — https://github.com/engal/MetaLAFFA
- **DIAMOND** (Produces the read-to-gene alignments whose filtered hits are the input to fractional counting)
- **Snakemake** (Pipeline framework that orchestrates the gene counting step alongside the other MetaLAFFA steps) — https://github.com/borenstein-lab/MetaLAFFA

## Evaluation signals

- Output TSV exists and is tab-delimited with one row per UniRef90 gene and samples as columns.
- Each sample's total counts are consistent with fractional (not double-counted) read contributions: a read tied across n genes adds exactly 1/n to each gene.
- Gene counts are length-normalized using the .norm file factors (gene length / average gene length), so raw hit sums and final abundances differ predictably.
- Counts derive only from retained best-e-value hits; counts computed before hit filtering should be higher.

## Limitations

- Fractional counting is MetaLAFFA's default, but splitting tied reads evenly assumes equal-quality matches genuinely represent equal contribution; distinct abundances are not resolved by read-level evidence.
- Requires a gene length normalization table; without it, longer genes are over-counted because they are more likely to be sequenced.
- Databases and sample alignment percentages vary (UniRef90 match rates ranged 53%–78% across samples in the paper), so gene coverage differs between samples.

## Evidence

- [intro] gene abundance calculation with fractional counts: "gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [methods] custom Python script implements fractional counting and length normalization: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [methods] rationale for gene length normalization: "the counts of reads mapping to a gene should be normalized by the length of the gene"
- [methods] input to gene counting is best e-value filtered hits: "keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations."
