---
name: shotgun-metagenome-vs-predicted-metagenome-comparison
description: Use when 16S OTU tables and matched shotgun metagenome KO tables are
  available for the same samples and you need to quantify at what sequencing depth
  PICRUSt prediction rivals direct metagenomic sequencing.
when_to_use_negative:
- No matched shotgun metagenomic data exist for the samples — accuracy benchmarking
  against observed KO abundances is impossible
- The community contains substantial viral or eukaryotic components — PICRUSt only
  infers bacterial and archaeal contributions, so the comparison would be invalid
- The 16S data are not OTU-picked against Greengenes+IMG reference (e.g. ASV-based
  or de novo picking), so PICRUSt copy-number normalization and per-genome KO lookup
  cannot be applied as described
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: PICRUSt
  role: Predicts KO metagenomes from the normalized 16S OTU table for comparison against
    observed metagenomes
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to
    produce the PICRUSt-compatible OTU table
- name: uclust
  role: Reference OTU picker against the Greengenes+IMG combined reference
- name: HUMAnN v0.98
  role: Annotates WGS reads to KOs to produce the observed metagenome KO abundance
    table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:6509ac5e88867b4f94d794d9a38e13e450c84ef54aa8bc830714a85d5d09f88e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HUMAnN
  - HUMAnN v0.98
  - PICRUSt
  - QIIME 1.5.0-dev
  - uclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# shotgun-metagenome-vs-predicted-metagenome-comparison

## Summary

Benchmarks PICRUSt-predicted KO abundance profiles (built from 16S OTU tables) against rarefied shallow shotgun metagenomic KO profiles using Spearman rank correlation across sequencing depths, to determine the read depth below which 16S-based prediction outperforms direct sequencing.

## When to use

Use when 16S OTU tables and matched shotgun metagenome KO tables are available for the same samples and you need to quantify at what sequencing depth PICRUSt prediction rivals direct metagenomic sequencing.

## When NOT to use

- No matched shotgun metagenomic data exist for the samples — accuracy benchmarking against observed KO abundances is impossible
- The community contains substantial viral or eukaryotic components — PICRUSt only infers bacterial and archaeal contributions, so the comparison would be invalid
- The 16S data are not OTU-picked against Greengenes+IMG reference (e.g. ASV-based or de novo picking), so PICRUSt copy-number normalization and per-genome KO lookup cannot be applied as described

## Inputs

- 16S OTU table built via closed-reference OTU picking (QIIME 1.5.0-dev, uclust against Greengenes+IMG)
- Shotgun metagenome KO abundance table annotated with HUMAnN v0.98
- PICRUSt pre-calculated 16S copy number and per-genome KO content tables

## Outputs

- spearman_vs_depth.csv — table of Spearman rank correlation vs sequencing depth
- spearman_vs_depth.png — comparison figure marking the 10^5-read convergence point and the ~72,000-read crossover

## How to apply

Build the 16S OTU table with closed-reference OTU picking (QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py, uclust against Greengenes+IMG) and annotate the shotgun metagenome to KOs with HUMAnN v0.98. Run PICRUSt: normalize the OTU table by predicted 16S copy number, then multiply normalized OTU abundance by each KO abundance in the genome and sum per sample to obtain predicted metagenomes. Rarefy both the 16S OTU table and the observed KO table at matched depths spanning shallow to ~10^5 assigned reads, randomly selecting a fixed number of counts per sample at each depth. At each depth compute the Spearman rank correlation between PICRUSt-predicted and rarefied observed KO abundances per sample. Account for gene annotation survival (~17.3%) and OTU-picking rates (~68.9%): the article's calibration found ~15,000 annotated metagenomic sequences matched PICRUSt's accuracy from 105 assigned 16S reads, yielding a crossover at ~72,000 total sequences below which PICRUSt outperforms shallow sequencing (r=0.82 at 10^5 reads vs r=0.18 for the shallow metagenome).

## Related tools

- **PICRUSt** (Predicts KO metagenomes from the normalized 16S OTU table for comparison against observed metagenomes) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to produce the PICRUSt-compatible OTU table)
- **uclust** (Reference OTU picker against the Greengenes+IMG combined reference)
- **HUMAnN v0.98** (Annotates WGS reads to KOs to produce the observed metagenome KO abundance table)

## Evaluation signals

- Spearman r at the deepest rarefaction depth (~10^5 reads) reaches ~0.82 between predicted and observed KO abundances
- Spearman r for the shallow rarefied metagenome is ~0.18, so predicted-vs-observed correlation should dominate at depths below ~72,000 total sequences
- Predicted metagenome per sample equals the sum of normalized OTU abundance times per-genome KO abundance, so totals must reflect 16S copy-number normalization
- Matched rarefaction depths are applied identically to both the OTU table and the observed KO table
- Crossover depth calculation must account for the 17.3% gene annotation survival and 68.9% OTU-picking rates

## Limitations

- PICRUSt does not infer viral or eukaryotic metagenome components, biasing the comparison for communities with such members
- Laterally transferred genes and some gene gain/loss events will be poorly predicted, with wide confidence intervals
- Prediction accuracy depends on availability of related reference genomes (quantified by NSTI); environments with phylogenetic gaps in the reference database reduce accuracy
- Strain-level variation cannot be distinguished when marker gene sequences are identical among strains

## Evidence
