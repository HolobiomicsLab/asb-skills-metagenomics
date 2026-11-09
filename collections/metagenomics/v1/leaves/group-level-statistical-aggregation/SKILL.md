---
name: group-level-statistical-aggregation
description: Use when LMAS per-reference metric tables (Contiguity, Multiplicity,
  Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) must be collapsed
  from individual-assembler to assembler-group level to test group-level performance
  claims.
when_to_use_negative:
- Input is raw FASTQ reads or assembler FASTA outputs rather than already-computed
  LMAS metric tables — run the LMAS workflow first
- The comparison must cover ABySS, MetaHipmer2, minia, or VelvetOptimiser — these
  were excluded from the article's subsequent analyses due to poor performance
- Goal is to compare assemblers at the individual-tool level (e.g. best assembler
  per sample) rather than test a group-level label effect
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Source of the per-reference metric tables (Contiguity, Validity, Lx, NAx,
    NGx, etc.) that are normalized and aggregated by group
  repo: https://github.com/B-UMMI/LMAS
- name: pandas
  role: Loading and group-wise aggregation of normalized per-reference scores into
    assembler_group_comparison.csv
- name: matplotlib
  role: Rendering per-reference group comparison plots as PNG figures in the style
    of Supplementary Figs S3-S5
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:2ddfee804ba4389fcd0a32b6080ff9ff3c436f1a0c0b8233d7470e37931cec4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - matplotlib
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# group-level-statistical-aggregation

## Summary

Aggregates per-assembler benchmarking metrics into predefined tool groups (genomic vs metagenomic assemblers) and compares group-level mean scores to test whether branding translates into performance differences on LMAS ZymoBIOMICS mock communities.

## When to use

Use when LMAS per-reference metric tables (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) must be collapsed from individual-assembler to assembler-group level to test group-level performance claims.

## When NOT to use

- Input is raw FASTQ reads or assembler FASTA outputs rather than already-computed LMAS metric tables — run the LMAS workflow first
- The comparison must cover ABySS, MetaHipmer2, minia, or VelvetOptimiser — these were excluded from the article's subsequent analyses due to poor performance
- Goal is to compare assemblers at the individual-tool level (e.g. best assembler per sample) rather than test a group-level label effect

## Inputs

- LMAS deposited per-reference metric tables for ZymoBIOMICS samples (evenly/logarithmically distributed, in silico and real) from Zenodo doi 10.5281/zenodo.7129554
- Per-reference metrics: Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx
- Assembler-to-group mapping: genomic = SKESA, SPAdes, Unicycler; metagenomic = GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes

## Outputs

- assembler_group_comparison.csv: aggregated mean normalized scores per group, sample, and reference replicon
- PNG figures of per-reference group comparisons (box/bar plots) in the style of Supplementary Figs S3-S5
- Consistency counts of whether the metagenomic group's mean normalized score exceeds the genomic group's across samples and references

## How to apply

Load the deposited LMAS global and per-reference metric tables for the ZymoBIOMICS samples (ENN, EMS, ERR2984773, LNN, LHS, ERR2935805; evenly and logarithmically distributed, in silico and real) with pandas, retaining per-reference metrics for contigs passing the 1 kbp minimum contig filter (default --minLength 1000). Normalize each metric per reference replicon across assemblers, using the paper's l_target=0.9 and n_target=0.5 as reference points. Partition assemblers into the genomic group (SKESA, SPAdes, Unicycler) and metagenomic group (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes), then aggregate mean normalized scores per group, sample, and reference replicon. Assess consistency by counting across samples and references whether the metagenomic group's mean exceeds the genomic group's, recording directions of differences. The rationale is that the article found no consistent group advantage, especially after filtering contigs <1 kbp, so the aggregation must isolate direction-of-difference counts rather than assume superiority of either group.

## Related tools

- **LMAS** (Source of the per-reference metric tables (Contiguity, Validity, Lx, NAx, NGx, etc.) that are normalized and aggregated by group) — https://github.com/B-UMMI/LMAS
- **pandas** (Loading and group-wise aggregation of normalized per-reference scores into assembler_group_comparison.csv)
- **matplotlib** (Rendering per-reference group comparison plots as PNG figures in the style of Supplementary Figs S3-S5)

## Examples

```
df.groupby(['group','sample','reference'])['normalized_score'].mean().reset_index().to_csv('assembler_group_comparison.csv')
```

## Evaluation signals

- Every assembler in the aggregated table maps to exactly one of the two groups (SKESA/SPAdes/Unicycler vs GATBMiniaPipeline/IDBA-UD/MEGAHIT/metaSPAdes) — no unmapped tools
- All aggregated contigs pass the 1 kbp minimum contig filter (default --minLength 1000)
- Reproduced pattern: metagenomic group means are not consistently higher than genomic group means across samples and reference replicons, particularly for contigs >1 kbp
- Group-mean output rows cover every combination of group x sample x reference replicon present in the per-reference input tables
- Figure set includes per-sample, per-reference group comparisons matching the metric set (Contiguity through NGx)

## Limitations

- ZymoBIOMICS mock community may not represent the metagenomic complexity of most researchers' samples, so group conclusions may not generalize
- Usability and performance findings are limited to default parameters; non-default parameter tuning was not explored
- Differences between groups tend to be attenuated after filtering for contigs <1 kbp, so conclusions depend on the contig-length cutoff applied

## Evidence

- [abstract] assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples.: "assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples."
- [results] no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [results] the differences between them tended to be attenuated after filtering for contigs <1 kbp: "the differences between them tended to be attenuated after filtering for contigs <1 kbp"
- [methods] minimum contig size as defined in the --minLength parameter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
