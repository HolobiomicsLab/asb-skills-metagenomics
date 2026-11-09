---
name: supplementary-figure-reproduction
description: Use when deposited LMAS metric tables (global and per-reference) for
  mock metagenomic samples must be re-analyzed and re-plotted to test whether metagenomic-branded
  assemblers consistently outperform genomic assemblers.
when_to_use_negative:
- The research question concerns individual assembler rankings or computational cost
  rather than genomic-vs-metagenomic group-level differences
- Data are single-isolate assemblies without a known reference replicon set, so no
  per-reference metrics (Contiguity, NGx, Breadth of coverage) can be computed
- Raw reads rather than LMAS output tables are the only available input, since this
  skill re-analyzes deposited metric tables, not assemblies
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LMAS
  role: Source of the deposited global and per-reference assembly evaluation metric
    tables that are aggregated and plotted
  repo: https://github.com/B-UMMI/LMAS
- name: pandas
  role: Loading, normalizing (l_target=0.9, n_target=0.5), and grouping the per-reference
    metric tables by assembler group, sample, and replicon
- name: matplotlib/plotly
  role: Rendering the per-reference group comparison box/bar figures analogous to
    Supplementary Figs S3–S5
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:6b9e0a68250dbf8f9110450879fb34350bd58ce2cdd1e969685293ee5219357b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - matplotlib/plotly
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# supplementary-figure-reproduction

## Summary

Reproduce the LMAS paper's supplementary group-comparison figures (Supplementary Figs S3–S5 style) by aggregating deposited per-reference assembly metrics over genomic (SKESA, SPAdes, Unicycler) versus metagenomic (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes) assembler groups on the ZymoBIOMICS mock communities.

## When to use

Use when deposited LMAS metric tables (global and per-reference) for mock metagenomic samples must be re-analyzed and re-plotted to test whether metagenomic-branded assemblers consistently outperform genomic assemblers.

## When NOT to use

- The research question concerns individual assembler rankings or computational cost rather than genomic-vs-metagenomic group-level differences
- Data are single-isolate assemblies without a known reference replicon set, so no per-reference metrics (Contiguity, NGx, Breadth of coverage) can be computed
- Raw reads rather than LMAS output tables are the only available input, since this skill re-analyzes deposited metric tables, not assemblies

## Inputs

- Deposited LMAS global metric tables (CSV) for ZymoBIOMICS samples (ENN, EMS, LNN, LHS, ERR2984773, ERR2935805)
- Deposited LMAS per-reference metric tables (CSV) per reference replicon, filtered to contigs >= 1 kbp
- Assembler-to-group mapping: genomic = SKESA, SPAdes, Unicycler; metagenomic = GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes

## Outputs

- assembler_group_comparison.csv with aggregated mean normalized scores per group, sample, and reference replicon
- PNG figures of per-reference group comparisons (box/bar plots of normalized scores per group per sample) in the style of Supplementary Figs S3–S5
- Consistency tally of whether the metagenomic group's mean normalized score exceeds the genomic group's across samples and references

## How to apply

Load the deposited LMAS global and per-reference metric tables (contigs passing the 1 kbp minimum contig filter, i.e. --minLength default 1000) for the ZymoBIOMICS samples (evenly/logarithmically distributed, in silico and real: ENN, EMS, LNN, LHS, ERR2984773, ERR2935805) with pandas. Normalize the per-reference metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) per reference replicon using the paper's l_target=0.9 and n_target=0.5 targets. Partition assemblers into the genomic group (SKESA, SPAdes, Unicycler) and the metagenomic group (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes), aggregating mean normalized scores per group, sample, and replicon. Plot box/bar comparisons with matplotlib/plotly in the style of Supplementary Figs S3–S5, then count across samples and references whether the metagenomic group mean exceeds the genomic group mean to assess consistency of the direction of differences.

## Related tools

- **LMAS** (Source of the deposited global and per-reference assembly evaluation metric tables that are aggregated and plotted) — https://github.com/B-UMMI/LMAS
- **pandas** (Loading, normalizing (l_target=0.9, n_target=0.5), and grouping the per-reference metric tables by assembler group, sample, and replicon)
- **matplotlib/plotly** (Rendering the per-reference group comparison box/bar figures analogous to Supplementary Figs S3–S5)

## Examples

```
import pandas as pd; df = pd.read_csv('lmas_per_reference_metrics.csv'); df = df[df['contig_len'] >= 1000]; groups = df.groupby(['assembler_group','sample','reference'])['Contiguity'].mean()
```

## Evaluation signals

- The direction of genomic-vs-metagenomic group differences is not consistently in favor of metagenomic assemblers across samples and reference replicons
- No significant group-level difference is observed after the 1 kbp contig filter (--minLength default 1000)
- The aggregated table assembler_group_comparison.csv contains one row per assembler-group x sample x reference replicon with mean normalized scores for all nine metrics
- Excluded assemblers (ABySS, MetaHipmer2, minia, VelvetOptimiser) are absent from the group comparison
- Normalization uses the paper's reference points l_target=0.9 and n_target=0.5 per reference replicon

## Limitations

- ZymoBIOMICS mock community may not represent the metagenomic complexity of most researchers' samples, limiting generalization of the reproduced figures
- Results reflect default assembler parameters only; non-default parameter tuning was not explored
- Findings may not generalize across community compositions since no assembler stood out as an undisputed all-purpose choice

## Evidence

- [results] no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [results] the differences between them tended to be attenuated after filtering for contigs <1 kbp: "the differences between them tended to be attenuated after filtering for contigs <1 kbp"
- [results] the workflow was run 3 times for all samples with default parameters: "the LMAS workflow was run 3 times for all samples using default parameters"
- [abstract] the workflow and its documentation are available at https://github.com/B-UMMI/LMAS and https://lmas.readthedocs.io/, respectively.: "the workflow and its documentation are available at https://github.com/B-UMMI/LMAS and https://lmas.readthedocs.io/, respectively."
- [discussion] The interactive report provides global and reference-specific performance metrics for exploration: "The interactive report provides an intuitive platform for data exploration, allowing the user to easily sift through global and reference-specific performance metrics for each sample"
