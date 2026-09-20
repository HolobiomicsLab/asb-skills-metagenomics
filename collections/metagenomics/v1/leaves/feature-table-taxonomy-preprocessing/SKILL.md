---
name: feature-table-taxonomy-preprocessing
description: Use when a raw 16S OTU count table (e.g. otu.tab.txt) and taxonomy table
  (tax.tab.txt) must be quality-controlled before survival-response microbiome analysis
  and the MiSurv default QC criteria need to be reproduced.
when_to_use_negative:
- The input is already the deposited post-QC data (biom.Rdata or the deposited post-QC
  otu.tab.txt/tax.tab.txt) — re-filtering would alter the feature/sample counts
- The analysis goal is raw read-level QC (Phred <20 trimming, chimera removal, 97%
  OTU picking) rather than feature-table-level filtering after OTU picking
- Survival data or analytic plans (survival time, censored/event, follow-up period)
  need processing rather than the abundance/taxonomy tables
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: MiSurv
  role: Web cloud platform whose Data Processing module implements the default QC
    criteria (kingdom, library size, mean proportion, taxonomic name) and outputs
    the filtered tables
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used to load, filter (subset_taxa/sample sums), and write the OTU
    and taxonomy tables when reconstructing the QC locally
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:6e53d9445bb5c00500128fc83cb0da39be323d2c98372c5c2dbddbe1cb073a4a
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# feature-table-taxonomy-preprocessing

## Summary

Reconstructs MiSurv's default microbiome quality control, which filters an OTU/feature count table and taxonomy table by kingdom annotation, sample library size, and mean relative abundance. On the example T1D gut microbiome dataset these defaults retain 294 features across 172 subjects.

## When to use

Use when a raw 16S OTU count table (e.g. otu.tab.txt) and taxonomy table (tax.tab.txt) must be quality-controlled before survival-response microbiome analysis and the MiSurv default QC criteria need to be reproduced.

## When NOT to use

- The input is already the deposited post-QC data (biom.Rdata or the deposited post-QC otu.tab.txt/tax.tab.txt) — re-filtering would alter the feature/sample counts
- The analysis goal is raw read-level QC (Phred <20 trimming, chimera removal, 97% OTU picking) rather than feature-table-level filtering after OTU picking
- Survival data or analytic plans (survival time, censored/event, follow-up period) need processing rather than the abundance/taxonomy tables

## Inputs

- Pre-QC OTU count table (otu.tab.txt, tab-delimited)
- Pre-QC taxonomy annotation table (tax.tab.txt, tab-delimited)
- Optionally the full phyloseq object biom.Rdata from the MiSurv Data folder

## Outputs

- Filtered abundance table (post_qc_otu_table.txt, tab-delimited; 294 features x 172 subjects on the example dataset)
- Filtered taxonomy table (post_qc_tax_table.txt, tab-delimited)
- Post-QC phyloseq object equivalent to the deposited biom.Rdata

## How to apply

Load the pre-QC OTU count table (otu.tab.txt) and taxonomy table (tax.tab.txt) into R with the phyloseq package, or use MiSurv's Data Processing module which exposes quality controls for 'kingdom, library size, mean proportion, taxonomic name'. Apply four default filters in order: (1) keep only taxa annotated with kingdom = Bacteria, dropping rows with erroneous or missing taxonomic names; (2) drop samples whose total library size is below 3,000 reads; (3) remove taxa whose mean relative proportion across retained samples is below 0.002%. Write the filtered abundance and taxonomy tables as tab-delimited post_qc_otu_table.txt and post_qc_tax_table.txt. The rationale is that these defaults produced the deposited post-QC phyloseq object (biom.Rdata) underlying the paper's T1D example analysis.

## Related tools

- **MiSurv** (Web cloud platform whose Data Processing module implements the default QC criteria (kingdom, library size, mean proportion, taxonomic name) and outputs the filtered tables) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used to load, filter (subset_taxa/sample sums), and write the OTU and taxonomy tables when reconstructing the QC locally)

## Examples

```
otu.tab <- read.table("otu.tab.txt", sep="\t", header=TRUE); tax.tab <- read.table("tax.tab.txt", sep="\t", header=TRUE)
```

## Evaluation signals

- Filtered tables contain exactly 294 features and 172 subjects on the example T1D dataset
- Taxonomic breakdown matches 7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species
- All retained taxa have kingdom = Bacteria with no erroneous or missing taxonomic names
- Every retained sample has a total library size of at least 3,000 reads
- Filtered tables match the deposited post-QC data (biom.Rdata / otu.tab.txt / tax.tab.txt) from the MiSurv Data folder

## Limitations

- The default thresholds (kingdom = Bacteria, 3,000-read minimum library size, 0.002% mean proportion) are tuned to the example T1D dataset; other datasets may need different cutoffs
- No changelog or R package version numbers are provided for the local phyloseq-based reconstruction, so exact reproducibility depends on package versions
- Upstream read-level QC (retaining reads with >75% original length, removing reads with more than three consecutive low-quality bases at Phred <20, ChimeraSlayer chimera removal) is performed outside this feature-table-level QC step

## Evidence

- [methods] Filter taxa to keep only those annotated with kingdom = Bacteria, removing rows with erroneous or missing taxonomic names.: [evidence span withheld — anchor status: not_found]
- [methods] Drop samples whose total library size is below 3,000 reads.: [evidence span withheld — anchor status: not_found]
- [methods] Remove taxa whose mean relative proportion across retained samples is below 0.002%.: [evidence span withheld — anchor status: not_found]
- [readme] quality controls (kingdom, library size, mean proportion, taxonomic name): "quality controls (kingdom, library size, mean proportion, taxonomic name)"
- [discussion] the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database (63)"
- [readme] we stored the all the final processed data that we used in our paper: "we stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
