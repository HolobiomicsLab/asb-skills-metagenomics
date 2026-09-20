---
name: tabular-result-export
description: Use when per-dataset, per-method benchmark scores (e.g., Quast/metaQuast
  genome fraction percentages) have been computed and must be aggregated and persisted
  as a CSV with per-dataset rows plus summary statistics for verification.
when_to_use_negative:
- Input is not a per-method score table (e.g., raw sequencing FASTQ reads or assembled
  contig FASTA) — assembly and Quast/metaQuast evaluation must happen first.
- The comparison target is a metric other than per-dataset relative gain over a best-competitor
  baseline (e.g., N50 contig length rankings, where MEGAHIT leads).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Method whose genome-fraction results are the numerator of the per-dataset
    gain exported to the CSV
  repo: https://github.com/HaploKit/StrainXpress
- name: pandas
  role: DataFrame loading, per-dataset gain computation, and aggregation into mean/quartile
    statistics for CSV export
- name: scipy
  role: Statistical aggregation of per-dataset gains (quartiles) alongside pandas
- name: Quast
  role: Source of the genome fraction benchmark table that feeds the tabular export
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:bd483ceb0292381a2a042afcfe54f158b56cc63ad8b74bed0aeca9a9d121d897
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - IDBA-UD
  - MEGAHIT
  - Minia3
  - Quast
  - SPAdes
  - StrainXpress
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# tabular-result-export

## Summary

Export per-dataset benchmark results (e.g., genome-fraction gains of StrainXpress over IDBA-UD, SPAdes, Minia3, MEGAHIT) and their aggregate summary statistics (mean, Q1, median, Q3) to a structured CSV such as gain_summary.csv, enabling verification against reported values.

## When to use

Use when per-dataset, per-method benchmark scores (e.g., Quast/metaQuast genome fraction percentages) have been computed and must be aggregated and persisted as a CSV with per-dataset rows plus summary statistics for verification.

## When NOT to use

- Input is not a per-method score table (e.g., raw sequencing FASTQ reads or assembled contig FASTA) — assembly and Quast/metaQuast evaluation must happen first.
- The comparison target is a metric other than per-dataset relative gain over a best-competitor baseline (e.g., N50 contig length rankings, where MEGAHIT leads).

## Inputs

- Per-dataset, per-method genome fraction table from Quast/metaQuast benchmark evaluations (CSV/DataFrame)
- List of competitor assembler methods (IDBA-UD, SPAdes, Minia3, MEGAHIT)

## Outputs

- gain_summary.csv containing one gain row per dataset plus aggregate summary (mean, Q1, median, Q3 of percentage gains)

## How to apply

Load the per-dataset, per-method genome fraction table produced by Quast/metaQuast evaluations into a pandas DataFrame. For each dataset, take the maximum genome fraction among the state-of-the-art assemblers (IDBA-UD, SPAdes, Minia3, MEGAHIT) as the best competitor baseline, then compute StrainXpress's percentage gain as (StrainXpress_genome_fraction - best_competitor_genome_fraction) / best_competitor_genome_fraction * 100. Aggregate the per-dataset gains with pandas/scipy into mean, first quartile, median, and third quartile, and compare against the reported values (mean 26.75%, Q1 18.51%, median 26.60%, Q3 35.05%). Write the per-dataset gains plus the aggregate summary to gain_summary.csv so the output can be checked for existence and per-dataset row completeness.

## Related tools

- **StrainXpress** (Method whose genome-fraction results are the numerator of the per-dataset gain exported to the CSV) — https://github.com/HaploKit/StrainXpress
- **pandas** (DataFrame loading, per-dataset gain computation, and aggregation into mean/quartile statistics for CSV export)
- **scipy** (Statistical aggregation of per-dataset gains (quartiles) alongside pandas)
- **Quast** (Source of the genome fraction benchmark table that feeds the tabular export)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('genome_fractions.csv'); comp=['IDBA-UD','SPAdes','Minia3','MEGAHIT']; best=df[comp].max(axis=1); g=(df['StrainXpress']-best)/best*100; print(g.mean(), g.quantile([.25,.5,.75])); g.to_csv('gain_summary.csv')"
```

## Evaluation signals

- Computed mean of per-dataset gains matches the reported 26.75%.
- Computed Q1, median, and Q3 match the reported 18.51%, 26.60%, and 35.05%, respectively.
- gain_summary.csv exists and contains one gain row per benchmark dataset (e.g., low/medium/high complexity synthetic sets, Bmock12, NWCs, Gut Metagenome, spike-in Salmonella sets).
- Every per-dataset gain is computed against the maximum genome fraction among IDBA-UD, SPAdes, Minia3, and MEGAHIT, not a single fixed competitor.

## Limitations

- No software version numbers are stated for any of the compared tools, so exported gains are not pinned to specific assembler releases (reproducibility gap).
- Genome fraction gains reflect the amount of strain-specific sequence reconstructed, not contig length quality — MEGAHIT outperforms all methods on N50, which this export does not capture.
- Short Illumina read length constrains contig lengths, so exported genome-fraction gains do not imply full-length strain-specific contigs.

## Evidence

- [abstract] The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets (first quartile: 18.51%, median: 26.60%, third: "The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets (first quartile: 18.51%, median: 26.60%, third"
- [results] Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40): "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [intro] A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9).: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [readme] conda install -c bioconda python=3.6 scipy pandas minimap2: "conda install -c bioconda python=3.6 scipy pandas minimap2"
