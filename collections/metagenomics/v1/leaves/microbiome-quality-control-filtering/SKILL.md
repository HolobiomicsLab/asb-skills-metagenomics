---
name: microbiome-quality-control-filtering
description: Use when raw OTU count tables and taxonomy tables (e.g., otu.tab.txt
  / tax.tab.txt from phyloseq) must be quality-controlled before survival-microbiome
  analysis and you need MiSurv's default filtering criteria reproduced.
when_to_use_negative:
- Input is already a post-QC feature table — the deposited post-QC data (biom.Rdata
  / otu.tab.txt / tax.tab.txt) already reflect these filters
- Input is raw sequencing reads (FASTQ) — this skill operates on OTU count and taxonomy
  tables, not on read-level QC such as the >75% length / <3 consecutive low-quality-base
  (Phred <20) trimming done upstream in QIIME 2.0
- The analysis target is continuous or binary responses — use MiCloud rather than
  MiSurv's survival-oriented processing module
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MiSurv
  role: Web platform whose Data Processing module implements the kingdom, library
    size, mean proportion, and taxonomic name quality controls being reproduced
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used to load the OTU and taxonomy tables and perform the taxa/samples
    filtering steps in R
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:efe61ab07b511d5e2ca2683c10cc89eb1a2fe900d40de83b992996989b19f4d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# microbiome-quality-control-filtering

## Summary

Reproduces MiSurv's default microbiome quality control pipeline — kingdom annotation, library-size, mean-proportion, and taxonomic-name filters — which, applied to the example Zhang et al. T1D gut microbiome dataset, retains 294 features across 172 subjects.

## When to use

Use when raw OTU count tables and taxonomy tables (e.g., otu.tab.txt / tax.tab.txt from phyloseq) must be quality-controlled before survival-microbiome analysis and you need MiSurv's default filtering criteria reproduced.

## When NOT to use

- Input is already a post-QC feature table — the deposited post-QC data (biom.Rdata / otu.tab.txt / tax.tab.txt) already reflect these filters
- Input is raw sequencing reads (FASTQ) — this skill operates on OTU count and taxonomy tables, not on read-level QC such as the >75% length / <3 consecutive low-quality-base (Phred <20) trimming done upstream in QIIME 2.0
- The analysis target is continuous or binary responses — use MiCloud rather than MiSurv's survival-oriented processing module

## Inputs

- Pre-QC OTU count table (otu.tab.txt, tab-delimited)
- Pre-QC taxonomy table (tax.tab.txt, tab-delimited)
- Optionally the phyloseq object biom.Rdata from the MiSurv Data folder

## Outputs

- post_qc_otu_table.txt (filtered abundance table, tab-delimited)
- post_qc_tax_table.txt (filtered taxonomy table, tab-delimited)
- Expected reference result: 294 features (7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species) across 172 subjects

## How to apply

Load the pre-QC OTU count table (otu.tab.txt) and taxonomy table (tax.tab.txt) into R via the phyloseq package. Filter taxa to keep only those annotated with kingdom = Bacteria, dropping rows with erroneous or missing taxonomic names (one of MiSurv's four documented QC controls: kingdom, library size, mean proportion, taxonomic name). Drop samples whose total library size is below 3,000 reads, then remove taxa whose mean relative proportion across retained samples is below 0.002%. Write the filtered abundance and taxonomy tables as tab-delimited post_qc_otu_table.txt and post_qc_tax_table.txt. The rationale is that erroneous annotations and tiny libraries/low-prevalence taxa add noise to the downstream data processing module that feeds MiSurv's four analytic modules.

## Related tools

- **MiSurv** (Web platform whose Data Processing module implements the kingdom, library size, mean proportion, and taxonomic name quality controls being reproduced) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used to load the OTU and taxonomy tables and perform the taxa/samples filtering steps in R)

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- Filtered tables contain exactly 294 features and 172 subjects
- Taxonomic breakdown of retained features matches 7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species
- Outputs match the deposited post-QC data (biom.Rdata / otu.tab.txt / tax.tab.txt) from the MiSurv Data folder
- No retained sample has total library size below 3,000 reads
- No retained taxon has mean relative proportion below 0.002% or a non-Bacteria/missing kingdom annotation

## Limitations

- The exact default QC criteria (kingdom=Bacteria, >=3,000 reads, >=0.002% mean proportion) were validated on the example T1D gut microbiome dataset; other datasets may require different thresholds
- Version numbers for the R packages used (e.g., phyloseq) are not documented, so exact reproducibility may depend on package versions
- MiSurv's Data Processing module also adjusts the follow-up period during QC, which this filtering-only skill does not cover

## Evidence

- [readme] quality controls (kingdom, library size, mean proportion, taxonomic name): "quality controls (kingdom, library size, mean proportion, taxonomic name)"
- [readme] we stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format"
- [discussion] we added the quality control to adjust the follow-up period: "we added the quality control to adjust the follow-up period"
