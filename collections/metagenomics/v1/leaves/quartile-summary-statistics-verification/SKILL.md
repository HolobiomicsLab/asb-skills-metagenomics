---
name: quartile-summary-statistics-verification
description: Use when a published benchmark summary statistic (e.g. mean or quartile
  gain over baselines) must be independently recomputed and verified from per-dataset
  evaluation tables such as Quast/metaQuast genome-fraction reports.
when_to_use_negative:
- Input data are raw sequencing reads or assemblies rather than a completed per-method
  evaluation table (genome fractions must be computed with Quast/metaQuast first)
- The claim to verify is not a distributional summary statistic over datasets but
  a per-dataset result already fully reported
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Method under test whose genome fraction gains over baselines are aggregated
    and verified
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: State-of-the-art baseline assembler contributing genome fraction values to
    the best-competitor baseline
- name: IDBA-UD
  role: State-of-the-art baseline assembler contributing genome fraction values to
    the best-competitor baseline
- name: Minia3
  role: State-of-the-art baseline assembler contributing genome fraction values to
    the best-competitor baseline
- name: MEGAHIT
  role: State-of-the-art baseline assembler contributing genome fraction values to
    the best-competitor baseline
- name: pandas
  role: Used to load the evaluation table and aggregate per-dataset gains (installed
    via conda in the StrainXpress environment)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:890a046519a872094c867b7d38636737885facf93b2ca02c46586bb8aa2fd248
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
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quartile-summary-statistics-verification

## Summary

Reproduces the StrainXpress headline benchmark claim that reconstructed strain-specific sequence exceeds the best state-of-the-art assembler by on average 26.75% (Q1 18.51%, median 26.60%, Q3 35.05%) by computing per-dataset genome-fraction gains from Quast/metaQuast evaluation tables and aggregating them with pandas/scipy.

## When to use

Use when a published benchmark summary statistic (e.g. mean or quartile gain over baselines) must be independently recomputed and verified from per-dataset evaluation tables such as Quast/metaQuast genome-fraction reports.

## When NOT to use

- Input data are raw sequencing reads or assemblies rather than a completed per-method evaluation table (genome fractions must be computed with Quast/metaQuast first)
- The claim to verify is not a distributional summary statistic over datasets but a per-dataset result already fully reported

## Inputs

- Per-dataset, per-method genome fraction table from Quast/metaQuast benchmark evaluations (pandas DataFrame)
- List of state-of-the-art baseline assembler methods: IDBA-UD, SPAdes, Minia3, MEGAHIT
- StrainXpress genome fraction values per benchmark dataset

## Outputs

- gain_summary.csv containing per-dataset StrainXpress gain rows plus aggregate summary (mean, Q1, median, Q3)
- Verification result comparing computed statistics to reported values (26.75%, 18.51%, 26.60%, 35.05%)

## How to apply

Load the per-dataset, per-method genome fraction table produced by Quast/metaQuast evaluation of the StrainXpress paper's benchmark datasets (low/medium/high complexity synthetic sets, Bmock12, NWCs, Gut Metagenome, Salmonella spike-ins) into a pandas DataFrame. For each dataset, take the maximum genome fraction among the state-of-the-art baselines IDBA-UD, SPAdes, Minia3 and MEGAHIT as the best competitor value. Compute the per-dataset percentage gain of StrainXpress as (StrainXpress_genome_fraction - best_competitor_genome_fraction) / best_competitor_genome_fraction * 100. Aggregate the gains using pandas/scipy to obtain the mean and the first quartile, median, and third quartile. Compare computed values against the reported summary statistics (mean 26.75%, Q1 18.51%, median 26.60%, Q3 35.05%) and record the verification result. Write the per-dataset gains plus aggregate summary to gain_summary.csv.

## Related tools

- **StrainXpress** (Method under test whose genome fraction gains over baselines are aggregated and verified) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (State-of-the-art baseline assembler contributing genome fraction values to the best-competitor baseline)
- **IDBA-UD** (State-of-the-art baseline assembler contributing genome fraction values to the best-competitor baseline)
- **Minia3** (State-of-the-art baseline assembler contributing genome fraction values to the best-competitor baseline)
- **MEGAHIT** (State-of-the-art baseline assembler contributing genome fraction values to the best-competitor baseline)
- **pandas** (Used to load the evaluation table and aggregate per-dataset gains (installed via conda in the StrainXpress environment))

## Examples

```
import pandas as pd; df = pd.read_csv('genome_fractions.csv'); base = df[['IDBA-UD','SPAdes','Minia3','MEGAHIT']].max(axis=1); gains = (df['StrainXpress'] - base)/base*100; print(gains.mean(), gains.quantile([0.25,0.5,0.75]))
```

## Evaluation signals

- Computed mean of per-dataset gains equals the reported 26.75%
- Computed first quartile, median, and third quartile match the reported 18.51%, 26.60%, and 35.05% respectively
- gain_summary.csv exists and contains one gain row per benchmark dataset plus the aggregate summary
- Every per-dataset gain is computed against the maximum genome fraction among IDBA-UD, SPAdes, Minia3 and MEGAHIT, not a single fixed baseline
- All gain values are finite percentages (best competitor genome fraction is non-zero)

## Limitations

- No software version numbers are stated for StrainXpress or any compared tool, so exact reproduction may differ across versions
- No named datasets or accessions for the benchmarking experiments are given in the benchmarking section, hindering retrieval of raw inputs
- Quast/metaQuast genome fractions must already exist; the skill does not cover the assembly or evaluation steps themselves

## Evidence

- [abstract] Mean gain matches abstract claim: "The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets"
- [intro] Baselines enumerated: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [results] Quast used for evaluation quantities: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [results] metaQuast for spike-in evaluation: "evaluated the quality of the assemblies of the 10 spiked in salmonella strains using metaQuast"
- [readme] Conda environment includes pandas: "conda install -c bioconda python=3.6 scipy pandas minimap2"
