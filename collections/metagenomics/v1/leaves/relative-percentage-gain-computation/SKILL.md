---
name: relative-percentage-gain-computation
description: Use when a benchmark table reports a per-dataset quality metric (e.g.,
  genome fraction) for a new method and several baselines, and the goal is to quantify
  how much the new method exceeds the baseline per dataset and across all datasets.
when_to_use_negative:
- The input does not contain per-dataset metric values for both the proposed method
  and at least one competitor baseline, so a relative gain cannot be formed.
- The claim to verify is an absolute metric (e.g., Genome Fraction 93.45% vs 74.74%
  for SPAdes) rather than a relative improvement over the best competitor.
- The metric distribution is better summarized by per-dataset pairwise rankings than
  by relative differences from the best baseline.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: StrainXpress
  role: Method whose genome fraction is the numerator of the per-dataset relative
    gain
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: State-of-the-art assembler whose genome fraction contributes to the best-competitor
    baseline
- name: IDBA-UD
  role: State-of-the-art assembler baseline in the per-dataset maximum
- name: Minia3
  role: State-of-the-art assembler baseline in the per-dataset maximum
- name: MEGAHIT
  role: State-of-the-art assembler baseline in the per-dataset maximum
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:dad837d50fa55c719a9caea42d282d12aab4dc7d2a819454731b747a40f7f375
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - IDBA-UD
  - MEGAHIT
  - Minia3
  - SPAdes
  - StrainXpress
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# relative-percentage-gain-computation

## Summary

Computes per-dataset relative percentage gains of a proposed method (StrainXpress) over the best state-of-the-art competitor on a quality metric (genome fraction from Quast/metaQuast), then aggregates gains into mean, first quartile, median, and third quartile summary statistics to reproduce headline claims such as an average 26.75% excess in reconstructed strain-specific sequence.

## When to use

Use when a benchmark table reports a per-dataset quality metric (e.g., genome fraction) for a new method and several baselines, and the goal is to quantify how much the new method exceeds the best baseline per dataset and across all datasets.

## When NOT to use

- The input does not contain per-dataset metric values for both the proposed method and at least one competitor baseline, so a relative gain cannot be formed.
- The claim to verify is an absolute metric (e.g., Genome Fraction 93.45% vs 74.74% for SPAdes) rather than a relative improvement over the best competitor.
- The metric distribution is better summarized by per-dataset pairwise rankings than by relative differences from the best baseline.

## Inputs

- Per-dataset, per-method genome fraction table from Quast/metaQuast benchmark evaluations (StrainXpress plus IDBA-UD, SPAdes, Minia3, MEGAHIT)
- List of state-of-the-art competitor method names to use as baselines

## Outputs

- gain_summary.csv containing per-dataset relative gain rows and aggregate statistics (mean 26.75%, Q1 18.51%, median 26.60%, Q3 35.05%)

## How to apply

Load the per-dataset, per-method genome fraction table produced by Quast/metaQuast evaluations into a pandas DataFrame. For each dataset, take the maximum genome fraction among the state-of-the-art assemblers (IDBA-UD, SPAdes, Minia3, MEGAHIT) as the best competitor baseline. Compute the relative gain of StrainXpress per dataset as (StrainXpress_genome_fraction − best_competitor_genome_fraction) / best_competitor_genome_fraction × 100. Aggregate the per-dataset gains with pandas/scipy to obtain the mean (expected 26.75%), first quartile (18.51%), median (26.60%), and third quartile (35.05%), and write per-dataset gains plus aggregates to gain_summary.csv.

## Related tools

- **StrainXpress** (Method whose genome fraction is the numerator of the per-dataset relative gain) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (State-of-the-art assembler whose genome fraction contributes to the best-competitor baseline)
- **IDBA-UD** (State-of-the-art assembler baseline in the per-dataset maximum)
- **Minia3** (State-of-the-art assembler baseline in the per-dataset maximum)
- **MEGAHIT** (State-of-the-art assembler baseline in the per-dataset maximum)

## Examples

```
import pandas as pd; gains = (df['StrainXpress'] - df[['IDBA-UD','SPAdes','Minia3','MEGAHIT']].max(axis=1)) / df[['IDBA-UD','SPAdes','Minia3','MEGAHIT']].max(axis=1) * 100
```

## Evaluation signals

- Computed mean of per-dataset gains matches the reported 26.75%.
- Computed first quartile, median, and third quartile match 18.51%, 26.60%, and 35.05% respectively.
- gain_summary.csv exists and contains one gain row per benchmark dataset (e.g., low/medium/high complexity synthetic sets, Bmock12, NWCs, Gut Metagenome, Salmonella spike-ins).
- Every per-dataset gain value is derived from the max genome fraction among IDBA-UD, SPAdes, Minia3, and MEGAHIT, not a fixed single competitor.
- All gain values are expressed as relative percentages (×100) rather than absolute genome-fraction percentage points.

## Limitations

- No software version numbers are stated for any of the compared tools, so exact reproduction of the genome-fraction table may vary across assembler versions.
- No named datasets or accessions for the benchmarking experiments are given in the relevant sections, hindering independent reassembly of the input metrics.

## Evidence

- [abstract] Headline gain statistic: "The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets (first quartile: 18.51%, median: 26.60%, third"
- [results] Evaluation metric provenance: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [intro] Baseline assembler set: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [readme] Dependency stack for the analysis environment: "conda install -c bioconda python=3.6 scipy pandas minimap2"
