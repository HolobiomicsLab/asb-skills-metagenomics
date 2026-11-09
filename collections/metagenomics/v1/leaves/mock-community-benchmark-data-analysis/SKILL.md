---
name: mock-community-benchmark-data-analysis
description: Use when deposited LMAS metric tables (global and per-reference) from
  mock metagenomic communities are available and the goal is to test whether assembler
  branding (genomic vs metagenomic) predicts performance on complex samples.
when_to_use_negative:
- Input is raw sequencing reads or assembled contigs rather than already-computed
  LMAS metric tables — run the LMAS workflow first
- 'The question is about strain-level (SNP-level) resolution: the paper found meaningful
  strain resolution was not achieved even by the best assemblers, so metric aggregation
  cannot answer it'
- 'Assemblies include contigs < 1 kbp in scope: the paper''s genomic vs metagenomic
  conclusion specifically concerns comparisons after the 1 kbp filter'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Source of the global and per-reference assembly evaluation metrics being analyzed;
    integrates assembly and evaluation into one pipeline
  repo: https://github.com/B-UMMI/LMAS
- name: LMAS.js
  role: Interactive HTML report generator for exploring global and reference replicon-specific
    LMAS metrics
  repo: https://github.com/B-UMMI/LMAS.js
- name: pandas
  role: Loading and normalizing the deposited LMAS metric tables and aggregating scores
    per assembler group
- name: matplotlib/plotly
  role: Plotting per-reference genomic vs metagenomic group comparisons
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:7cb182e74b01249783007c376c4bea853b28aabde45d1b1791ff3866b993fe95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - LMAS.js
  - matplotlib/plotly
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mock-community-benchmark-data-analysis

## Summary

Analysis of LMAS benchmark output tables on defined mock communities (e.g. ZymoBIOMICS evenly/logarithmically distributed, in silico and real samples) to compare genomic versus metagenomic de novo assembler performance using global and per-reference normalized metrics.

## When to use

Use when deposited LMAS metric tables (global and per-reference) from mock metagenomic communities are available and the goal is to test whether assembler branding (genomic vs metagenomic) predicts performance on complex samples.

## When NOT to use

- Input is raw sequencing reads or assembled contigs rather than already-computed LMAS metric tables — run the LMAS workflow first
- The question is about strain-level (SNP-level) resolution: the paper found meaningful strain resolution was not achieved even by the best assemblers, so metric aggregation cannot answer it
- Assemblies include contigs < 1 kbp in scope: the paper's genomic vs metagenomic conclusion specifically concerns comparisons after the 1 kbp filter

## Inputs

- LMAS global metrics tables for ZymoBIOMICS and other mock community samples (ENN, EMS, ERR2984773, LNN, LHS, ERR2935805, BMock12, Gut-Mix-RR, Gut-Mix-HiLo)
- LMAS per-reference metric tables keyed by reference replicon, for contigs >= 1000 bp (--minLength default 1000)
- Assembler grouping labels (genomic: SKESA, SPAdes, Unicycler; metagenomic: GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes)
- Normalization targets l_target=0.9 and n_target=0.5 from the paper

## Outputs

- aggregated assembler_group_comparison.csv with mean normalized scores per assembler group, sample, and reference replicon
- PNG figures of per-reference group comparisons (box/bar plots of normalized scores per group per sample), in the style of Supplementary Figs S3-S5
- Consistency count of samples/references where the metagenomic group's mean normalized score exceeds the genomic group's

## How to apply

Load the deposited LMAS global and per-reference metric tables for the ZymoBIOMICS samples (ENN/EMS/ERR2984773 evenly distributed, LNN/LHS/ERR2935805 logarithmically distributed) with pandas, retaining per-reference metrics for contigs passing the --minLength default 1000 filter. Normalize the per-reference metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) per reference replicon across assemblers, using the paper's l_target=0.9 and n_target=0.5 targets as reference points. Partition assemblers into genomic (SKESA, SPAdes, Unicycler) and metagenomic (GATBMiniaPipeline, IDBA-UD, MEGAHIT, metaSPAdes) groups, excluding ABySS, MetaHipmer2, minia, and VelvetOptimiser as the paper did, then aggregate mean normalized scores per group, sample, and reference replicon and assess across samples whether the metagenomic group's mean exceeds the genomic group's consistently. The expected result, per the paper, is that differences are not consistently in the metagenomic assemblers' favor, particularly after the 1 kbp contig filter.

## Related tools

- **LMAS** (Source of the global and per-reference assembly evaluation metrics being analyzed; integrates assembly and evaluation into one pipeline) — https://github.com/B-UMMI/LMAS
- **LMAS.js** (Interactive HTML report generator for exploring global and reference replicon-specific LMAS metrics) — https://github.com/B-UMMI/LMAS.js
- **pandas** (Loading and normalizing the deposited LMAS metric tables and aggregating scores per assembler group)
- **matplotlib/plotly** (Plotting per-reference genomic vs metagenomic group comparisons)

## Examples

```
import pandas as pd; df = pd.read_csv('per_reference_metrics.csv'); gen=['SKESA','SPAdes','Unicycler']; meta=['GATBMiniaPipeline','IDBA-UD','MEGAHIT','metaSPAdes']
```

## Evaluation signals

- Excluding ABySS, MetaHipmer2, minia, and VelvetOptimiser matches the paper's reported exclusion of poor performers from subsequent analyses
- All per-reference metrics are computed only for contigs >= 1000 bp (the --minLength default), reproducing the filter under which the paper found no significant genomic vs metagenomic difference
- Normalized scores use the paper's l_target=0.9 and n_target=0.5 reference points; check normalization is per reference replicon across assemblers
- Direction-of-difference counts across samples and replicons should show no consistent metagenomic-group advantage, reproducing the reported pattern
- Output CSV rows should cover all group × sample × reference-replicon combinations present in the input tables

## Limitations

- Only default assembler parameters were benchmarked, so conclusions may not hold under non-default parameter tuning
- ZymoBIOMICS and other mock communities may not represent the metagenomic complexity of most researchers' samples, limiting generalization
- No undisputed all-purpose assembler was identified; results may not generalize across community compositions
- LMAS runs are computationally intensive due to de novo assembly, and assembler software versions were not stated in the text

## Evidence

- [abstract] assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples.: "assemblers branded as metagenomic specific did not consistently outperform other genomic assemblers in metagenomic samples."
- [results] no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [methods] The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [discussion] LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools.: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [readme] LMAS is an automated workflow enabling the benchmarking of traditional and metagenomic prokaryotic de novo assembly software using defined mock communities.: [evidence span withheld — anchor status: not_found]
