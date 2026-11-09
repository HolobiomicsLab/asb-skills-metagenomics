---
name: genome-fraction-benchmark-analysis
description: Use when a Quast/metaQuast genome-fraction table comparing StrainXpress
  against IDBA-UD, SPAdes, Minia3 and MEGAHIT across benchmark metagenome data sets
  must be turned into per-dataset and aggregate gain statistics.
when_to_use_negative:
- Input is raw reads or assemblies rather than an existing Quast/metaQuast genome-fraction
  table — run the assemblies and Quast evaluation first (gain cannot be computed directly).
- The comparison does not include a StrainXpress column, since the gain is defined
  relative to StrainXpress's genome fraction.
- Only a single assembler's output is available, so no best-competitor baseline across
  IDBA-UD, SPAdes, Minia3 and MEGAHIT can be established.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Method under evaluation; its genome fraction per dataset is the numerator
    of the gain computation
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: State-of-the-art competitor assembler; contributes genome-fraction values
    to the best-competitor baseline
- name: IDBA-UD
  role: State-of-the-art competitor assembler; contributes genome-fraction values
    to the best-competitor baseline
- name: Minia3
  role: State-of-the-art competitor assembler; contributes genome-fraction values
    to the best-competitor baseline
- name: MEGAHIT
  role: State-of-the-art competitor assembler; contributes genome-fraction values
    to the best-competitor baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:39fd97661f44febfc6ae821dd9cf46fab7ce728d563b80796ecf843c9ed8d429
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# genome-fraction-benchmark-analysis

## Summary

Reproduce the headline benchmark result of StrainXpress by computing per-dataset genome-fraction gains over the best state-of-the-art assembler (IDBA-UD, SPAdes, Minia3, MEGAHIT) from Quast/metaQuast tables, yielding a mean gain of 26.75% with quartiles 18.51/26.60/35.05.

## When to use

Use when a Quast/metaQuast genome-fraction table comparing StrainXpress against IDBA-UD, SPAdes, Minia3 and MEGAHIT across benchmark metagenome data sets must be turned into per-dataset and aggregate gain statistics.

## When NOT to use

- Input is raw reads or assemblies rather than an existing Quast/metaQuast genome-fraction table — run the assemblies and Quast evaluation first (gain cannot be computed directly).
- The comparison does not include a StrainXpress column, since the gain is defined relative to StrainXpress's genome fraction.
- Only a single assembler's output is available, so no best-competitor baseline across IDBA-UD, SPAdes, Minia3 and MEGAHIT can be established.

## Inputs

- Per-dataset, per-method genome fraction table from Quast/metaQuast benchmark evaluations (covering StrainXpress, IDBA-UD, SPAdes, Minia3, MEGAHIT)
- Benchmark metagenome datasets (e.g. low complexity 20-strain/10-species, medium complexity 100-strain/30-species, high complexity 1057-strain/376-species, Bmock12, NWCs, Gut Metagenome, Salmonella spike-in)

## Outputs

- gain_summary.csv with per-dataset StrainXpress gain rows plus aggregate mean, Q1, median, Q3
- Verification record comparing computed statistics to reported values (mean 26.75%, Q1 18.51%, median 26.60%, Q3 35.05%)

## How to apply

Load the per-dataset, per-method genome fraction table produced by Quast/metaQuast evaluation into a pandas DataFrame. For each dataset, take the maximum genome fraction among the four state-of-the-art assemblers (IDBA-UD, SPAdes, Minia3, MEGAHIT) as the best competitor baseline. Compute the percentage gain of StrainXpress per dataset as (StrainXpress_genome_fraction - best_competitor_genome_fraction) / best_competitor_genome_fraction * 100. Aggregate the per-dataset gains with pandas/scipy to obtain the mean, first quartile, median and third quartile, then compare against the reported values of 26.75%, 18.51%, 26.60% and 35.05%. Write the per-dataset gains plus the aggregate summary to gain_summary.csv.

## Related tools

- **StrainXpress** (Method under evaluation; its genome fraction per dataset is the numerator of the gain computation) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (State-of-the-art competitor assembler; contributes genome-fraction values to the best-competitor baseline)
- **IDBA-UD** (State-of-the-art competitor assembler; contributes genome-fraction values to the best-competitor baseline)
- **Minia3** (State-of-the-art competitor assembler; contributes genome-fraction values to the best-competitor baseline)
- **MEGAHIT** (State-of-the-art competitor assembler; contributes genome-fraction values to the best-competitor baseline)

## Examples

```
python -c "import pandas as pd, scipy.stats as st; df=pd.read_csv('genome_fractions.csv'); base=df[['IDBA-UD','SPAdes','Minia3','MEGAHIT']].max(axis=1); gain=(df['StrainXpress']-base)/base*100; print(gain.mean(), gain.quantile([.25,.5,.75]))"
```

## Evaluation signals

- Computed mean of per-dataset gains equals the reported 26.75%.
- Computed first quartile, median and third quartile equal the reported 18.51%, 26.60% and 35.05% respectively.
- gain_summary.csv exists and contains one gain row per benchmark dataset.
- Per-dataset gains are non-negative wherever StrainXpress ranks first (e.g. low complexity 2x250 bp data: 93.45% vs 74.74% for SPAdes).
- All genome-fraction source values fall in the plausible 0-100% range and each dataset row has values for all five assemblers.

## Limitations

- No software version numbers are stated for StrainXpress or any competitor tool, constraining exact reproducibility of the benchmark.
- Results are tied to Quast/metaQuast genome fraction as the quality metric; other metrics (e.g. N50) may favor competitors — MEGAHIT outperforms all other methods in terms of contig length (N50 in particular).
- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs, which affects genome-fraction interpretation.

## Evidence

- [abstract] headline mean gain over best competitor: "The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets (first quartile: 18.51%, median: 26.60%, third"
- [intro] competitor assembler set: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [results] metric provenance: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [results] example single-dataset comparison: "StrainXpress: 93.45% versus 74.74% by SPAdes, as the second highest scoring"
- [readme] installation via conda (README): "conda install -c bioconda python=3.6 scipy pandas minimap2"
