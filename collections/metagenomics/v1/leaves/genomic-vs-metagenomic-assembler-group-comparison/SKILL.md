---
name: genomic-vs-metagenomic-assembler-group-comparison
description: Use when LMAS global/per-reference metric tables from mock metagenomic
  communities (e.g. ZymoBIOMICS evenly/logarithmically distributed, in silico or real)
  are available and the goal is to test whether assembler branding predicts performance.
when_to_use_negative:
- When no reference replicons are known (e.g. real uncharacterized samples) so per-reference
  metrics like NAx/NGx and Identity cannot be computed
- When the question is absolute assembler ranking rather than testing the genomic-vs-metagenomic
  labeling hypothesis
- When assemblies include contigs <1 kbp and the goal is the paper's no-significant-group-difference
  comparison, which is defined on the >1 kbp (--minLength 1000) filtered set
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Produces the global and per-reference assembly metric tables that are grouped
    and compared by assembler label
  repo: https://github.com/B-UMMI/LMAS
- name: SKESA
  role: Genomic assembler group member
- name: SPAdes
  role: Genomic assembler group member
- name: Unicycler
  role: Genomic assembler group member
- name: GATBMiniaPipeline
  role: Metagenomic assembler group member
- name: IDBA-UD
  role: Metagenomic assembler group member
- name: MEGAHIT
  role: Metagenomic assembler group member
- name: metaSPAdes
  role: Metagenomic assembler group member
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:133610a6883f746dcaeb4828d11b286aaec1e16aa880ac618f886336c9a99d69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - GATBMiniaPipeline
  - IDBA-UD
  - LMAS
  - MEGAHIT
  - SKESA
  - SPAdes
  - Unicycler
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# genomic-vs-metagenomic-assembler-group-comparison

## Summary

Benchmarks whether assemblers labeled 'metagenomic' (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes) consistently outperform 'genomic' assemblers (SKESA, SPAdes, Unicycler) on defined mock communities, using LMAS per-reference normalized metrics with a 1 kbp contig minimum (--minLength default 1000).

## When to use

Use when LMAS global/per-reference metric tables from mock metagenomic communities (e.g. ZymoBIOMICS evenly/logarithmically distributed, in silico or real) are available and the goal is to test whether assembler branding predicts performance.

## When NOT to use

- When no reference replicons are known (e.g. real uncharacterized samples) so per-reference metrics like NAx/NGx and Identity cannot be computed
- When the question is absolute assembler ranking rather than testing the genomic-vs-metagenomic labeling hypothesis
- When assemblies include contigs <1 kbp and the goal is the paper's no-significant-group-difference comparison, which is defined on the >1 kbp (--minLength 1000) filtered set

## Inputs

- LMAS global metric tables (CSV/TSV) for ZymoBIOMICS and other mock community samples (ENN, EMS, ERR2984773, LNN, LHS, ERR2935805, BMock12, Gut-Mix-RR, Gut-Mix-HiLo)
- LMAS per-reference metric tables aligned to known mock community reference replicons
- Assembler-to-group mapping: genomic (SKESA, SPAdes, Unicycler) vs metagenomic (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes)
- Normalization targets l_target=0.9 and n_target=0.5 and minimum contig length (--minLength default 1000)

## Outputs

- assembler_group_comparison.csv: aggregated mean normalized scores per assembler group, sample, and reference replicon
- Per-reference group comparison figures (PNG box/bar plots per sample) analogous to Supplementary Figs S3-S5
- Consistency tally of samples/references where metagenomic group mean exceeds genomic group mean, with direction of differences

## How to apply

Load LMAS global and per-reference metric tables for the mock samples (ENN, EMS, ERR2984773, LNN, LHS, ERR2935805) with pandas, retaining per-reference metrics for contigs passing the 1 kbp minimum contig filter (--minLength default 1000). Normalize per-reference metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) per reference replicon across assemblers using the paper's targets l_target=0.9 and n_target=0.5. Partition assemblers into genomic (SKESA, SPAdes, Unicycler) and metagenomic (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes) groups and aggregate mean normalized scores per group, sample, and reference replicon. Exclude poorly performing assemblers as the paper did (ABySS, MetaHipmer2, minia, VelvetOptimiser) and plot per-reference group comparisons (box/bar plots) in the style of Supplementary Figs S3-S5. Assess consistency by counting, across samples and references, whether the metagenomic group's mean normalized score exceeds the genomic group's; the expected result is that differences are not consistently in the metagenomic assemblers' favor, particularly after the 1 kbp filter, because individual assembler characteristics rather than the label determine performance.

## Related tools

- **LMAS** (Produces the global and per-reference assembly metric tables that are grouped and compared by assembler label) — https://github.com/B-UMMI/LMAS
- **SKESA** (Genomic assembler group member)
- **SPAdes** (Genomic assembler group member)
- **Unicycler** (Genomic assembler group member)
- **GATBMiniaPipeline** (Metagenomic assembler group member)
- **IDBA-UD** (Metagenomic assembler group member)
- **MEGAHIT** (Metagenomic assembler group member)
- **metaSPAdes** (Metagenomic assembler group member)

## Examples

```
import pandas as pd; df = pd.read_csv('per_reference_metrics.csv', sep='\t'); df.groupby(['group','sample','reference'])['score_norm'].mean().to_csv('assembler_group_comparison.csv')
```

## Evaluation signals

- After the 1 kbp contig filter, group-level differences between genomic and metagenomic assemblers are not consistently in favor of the metagenomic group across samples and reference replicons
- Excluded assemblers (ABySS, MetaHipmer2, minia, VelvetOptimiser) are absent from the aggregated comparison, matching the paper's exclusion of poor performers
- Normalized per-reference scores use the paper's targets l_target=0.9 and n_target=0.5 and metrics Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx
- The aggregated table contains rows per group x sample x reference replicon and saves cleanly to assembler_group_comparison.csv
- Comparison is run on LMAS outputs obtained from 3 replicate runs with default parameters per sample

## Limitations

- ZymoBIOMICS mock communities may not represent the metagenomic complexity of most researchers' samples, so group conclusions may not generalize
- Only default assembler parameters were tested; non-default parameter tuning was not explored, limiting conclusions about labeled groups
- Some assemblers fail on specific samples (e.g. ABySS failed to produce an assembly for ERR2984773 in 1 of the runs), so group aggregation may have unequal assembler coverage
- LMAS is computationally intensive due to de novo assembly; re-running to obtain new group comparisons is costly

## Evidence

- [abstract] assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples.: "assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples."
- [results] no significant difference between groups after 1 kbp filtering: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [results] excluded assemblers from subsequent analyses: "the following assemblers have not been included in subsequent analyses: ABySS, MetaHipmer2, minia, and VelvetOptimiser"
- [methods] minimum contig size parameter controls the contig length filter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [results] workflow run under default parameters three times: "the LMAS workflow was run 3 times for all samples using default parameters"
