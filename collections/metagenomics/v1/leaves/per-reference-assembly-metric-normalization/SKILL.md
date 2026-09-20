---
name: per-reference-assembly-metric-normalization
description: Use when comparing de novo assembler performance on mock metagenomic
  communities with known reference replicons, and per-reference metrics from LMAS
  global/per-reference tables must be placed on a comparable scale across assemblers.
when_to_use_negative:
- When no reference ground truth is available for the sample — per-reference normalization
  requires known reference replicons, so metagenomic samples without defined mock
  composition cannot be normalized this way
- When the analysis targets global (whole-assembly) metrics rather than replicon-specific
  performance; the paper's global tables do not need per-reference normalization
- When comparing assemblies that include contigs below the 1 kbp --minLength filter,
  since the reported genomic-vs-metagenomic pattern holds after filtering contigs
  <1 kbp
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Automated Nextflow workflow that assembles mock-community reads with multiple
    assemblers and produces the global and per-reference metric tables that this skill
    normalizes
  repo: https://github.com/B-UMMI/LMAS
- name: MetaQUAST
  role: Reference-based assembly evaluation approach whose concept of aligning contigs
    to a reference underpins the per-replicon metrics being normalized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:ac83821de4397f4b048292901d041cb348c9ac4b849df78a43043067ae25b3fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - MetaQUAST
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# per-reference-assembly-metric-normalization

## Summary

Normalize LMAS per-reference assembly metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) across assemblers for each reference replicon of defined mock communities, using the paper's targets l_target=0.9 and n_target=0.5, to enable fair cross-assembler comparisons in metagenomic samples.

## When to use

Use when comparing de novo assembler performance on mock metagenomic communities with known reference replicons, and per-reference metrics from LMAS global/per-reference tables must be placed on a comparable scale across assemblers.

## When NOT to use

- When no reference ground truth is available for the sample — per-reference normalization requires known reference replicons, so metagenomic samples without defined mock composition cannot be normalized this way
- When the analysis targets global (whole-assembly) metrics rather than replicon-specific performance; the paper's global tables do not need per-reference normalization
- When comparing assemblies that include contigs below the 1 kbp --minLength filter, since the reported genomic-vs-metagenomic pattern holds after filtering contigs <1 kbp

## Inputs

- LMAS deposited per-reference metric tables for mock community samples (ZymoBIOMICS evenly/logarithmically distributed, in silico and real: ERR2984773, ERR2935805, BMock12, Gut-Mix-RR, Gut-Mix-HiLo)
- Reference replicon definitions for the mock community (per-reference ground truth used by LMAS)
- Assembler-to-group mapping (genomic: SKESA, SPAdes, Unicycler; metagenomic: GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes)
- LMAS parameter values, notably --minLength (default 1000) and l_target=0.9, n_target=0.5

## Outputs

- Normalized per-reference metric table (one row per assembler x sample x reference replicon x metric)
- Aggregated group-level comparison table (e.g. assembler_group_comparison.csv) with mean normalized scores per group, sample, and replicon
- Per-reference group comparison plots (box/bar plots of normalized scores, styled after Supplementary Figs S3-S5) saved as PNG

## How to apply

Load the deposited LMAS global and per-reference metric tables (e.g. for ZymoBIOMICS evenly/logarithmically distributed, in silico and real samples such as ERR2984773 and ERR2935805) with pandas, keeping per-reference metrics for contigs passing the --minLength default of 1000 bp. Normalize each per-reference metric (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) per reference replicon across assemblers, using the paper's reference points l_target=0.9 and n_target=0.5 so scores are relative to the best achievable value per replicon. Optionally aggregate normalized scores by assembler group — genomic (SKESA, SPAdes, Unicycler) versus metagenomic (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes) — per sample and replicon. The rationale is that raw per-replicon metrics differ in scale and direction, so normalization is required before averaging or plotting group-level comparisons (as in Supplementary Figs S3-S5). Save the aggregated table (e.g. assembler_group_comparison.csv) and per-reference plots as PNG for downstream consistency checks.

## Related tools

- **LMAS** (Automated Nextflow workflow that assembles mock-community reads with multiple assemblers and produces the global and per-reference metric tables that this skill normalizes) — https://github.com/B-UMMI/LMAS
- **MetaQUAST** (Reference-based assembly evaluation approach whose concept of aligning contigs to a reference underpins the per-replicon metrics being normalized)

## Examples

```
import pandas as pd; df = pd.read_csv('per_reference_metrics.csv'); df = df[df['contig_length'] >= 1000]; df['norm_score'] = df.groupby(['sample','replicon','metric'])['score'].transform(lambda s: s / s.max())
```

## Evaluation signals

- All per-reference metric values fall in the normalized [0,1] range relative to the l_target=0.9 / n_target=0.5 reference points, with the best-performing assembler per replicon approaching 1
- Contigs shorter than the --minLength default of 1000 bp are excluded from every normalized row (row count matches the filtered assembly)
- Group means reproduce the paper's pattern: metagenomic-branded assemblers do NOT consistently exceed genomic assemblers across samples and replicons (no significant group-level difference after the 1 kbp filter)
- Each sample (ENN, EMS, ERR2984773, LNN, LHS, ERR2935805, BMock12, Gut-Mix-RR, Gut-Mix-HiLo) and reference replicon is represented in the aggregated table
-  assembler_group_comparison.csv contains one mean normalized score per assembler group x sample x replicon with no missing group entries

## Limitations

- Meaningful strain resolution at the SNP level was not achieved by any assembler, so per-reference metrics cannot capture strain-level accuracy
- ZymoBIOMICS and other mock communities may not represent the metagenomic complexity of most researchers' samples, limiting generalization of normalized comparisons
- Assemblers were benchmarked with default parameters only; normalized rankings may differ under non-default parameter tuning
- No single assembler is an undisputed all-purpose choice, so normalized scores should not be read as a universal ranking

## Evidence

- [abstract] per-reference normalization supports metric comparability: "assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples."
- [results] 1 kbp filter effect: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [intro] reference-based evaluation basis: "MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference"
- [methods] minLength parameter controls contig filter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [results] LMAS runs under default parameters: "the LMAS workflow was run 3 times for all samples using default parameters"
- [abstract] no universal assembler: "No single assembler appeared an ideal choice for short-read metagenomic prokaryote replicon assembly, each showing specific strengths."
