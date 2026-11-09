---
name: spearman-correlation-of-ko-abundance-profiles
description: Use when a 16S OTU table with a PICRUSt-predicted metagenome must be
  validated against a shotgun-annotated KO table (e.g. HUMAnN v0.98 KOs), especially
  across rarefied sequencing depths to find where prediction accuracy converges or
  crosses over.
when_to_use_negative:
- The community contains substantial viral or eukaryotic components, since the 16S-based
  prediction covers only bacterial and archaeal genomes
- Samples have high NSTI (no closely related reference genomes), where predicted KO
  profiles are expected to be inaccurate and correlation is uninformative
- The comparison goal is strain-level or laterally transferred gene content, which
  identical marker gene sequences and gain/loss events cannot resolve
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PICRUSt
  role: Normalizes the OTU table by 16S copy number predictions and predicts the KO
    abundance metagenome whose correlation with observed KOs is evaluated
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Generates the PICRUSt-compatible 16S OTU table via the closed-reference OTU
    picking protocol used as prediction input
- name: HUMAnN v0.98
  role: Annotates shotgun metagenomic reads to KOs, producing the observed KO table
    used as the correlation reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:547c720d0202afe494de6e09a1e1be86cf4c4b8898f5fc8db6c203fa69f717cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HUMAnN
  - HUMAnN v0.98
  - PICRUSt
  - QIIME 1.5.0-dev
  - uclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# spearman-correlation-of-ko-abundance-profiles

## Summary

Quantifies agreement between PICRUSt-predicted KO abundance profiles and observed (HUMAnN-annotated, rarefied) metagenomic KO profiles by computing per-sample Spearman rank correlations, optionally as a function of sequencing depth, to benchmark 16S-based metagenome prediction against shallow shotgun sequencing.

## When to use

Use when a 16S OTU table with a PICRUSt-predicted metagenome must be validated against a shotgun-annotated KO table (e.g. HUMAnN v0.98 KOs), especially across rarefied sequencing depths to find where prediction accuracy converges or crosses over.

## When NOT to use

- The community contains substantial viral or eukaryotic components, since the 16S-based prediction covers only bacterial and archaeal genomes
- Samples have high NSTI (no closely related reference genomes), where predicted KO profiles are expected to be inaccurate and correlation is uninformative
- The comparison goal is strain-level or laterally transferred gene content, which identical marker gene sequences and gain/loss events cannot resolve

## Inputs

- QIIME closed-reference 16S OTU table (uclust against Greengenes+IMG)
- PICRUSt-predicted KO abundance (metagenome) table per sample
- Shotgun metagenome KO abundance table annotated with HUMAnN v0.98
- Sequence of matched rarefaction depths spanning shallow reads to ~10^5 assigned reads

## Outputs

- Per-depth Spearman rank correlation values between predicted and observed KO profiles (spearman_vs_depth.csv)
- Plot of Spearman r versus sequencing depth with convergence and crossover points marked (spearman_vs_depth.png)

## How to apply

Build the PICRUSt-predicted metagenome by first normalizing the closed-reference QIIME (1.5.0-dev, uclust against Greengenes+IMG) OTU table by predicted 16S copy number, then multiplying normalized OTU abundance by each KO abundance in the genome and summing per sample. Rarefy both the OTU table and the observed KO table at matched depths (spanning shallow up to ~10^5 assigned reads), accounting for the annotation survival rate (17.3%) and OTU-picking rate (68.9%) when matching depth scales. At each depth, compute the Spearman rank correlation between predicted and rarefied observed KO abundances per sample. Tabulate r versus depth (e.g. spearman_vs_depth.csv), plot it, and read off the convergence point (~10^5 reads; r=0.82) and the crossover (~72,000 total sequences per sample) below which PICRUSt accuracy with ~105 assigned 16S reads matches ~15,000 annotated metagenomic sequences.

## Related tools

- **PICRUSt** (Normalizes the OTU table by 16S copy number predictions and predicts the KO abundance metagenome whose correlation with observed KOs is evaluated) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Generates the PICRUSt-compatible 16S OTU table via the closed-reference OTU picking protocol used as prediction input)
- **HUMAnN v0.98** (Annotates shotgun metagenomic reads to KOs, producing the observed KO table used as the correlation reference)

## Examples

```
pick_reference_otus_through_otu_table.py -i otu_table.biom -r gg_img_16S.fna -o closed_ref_otus/  # then predict KOs, rarefy both tables, and correlate per depth
```

## Evaluation signals

- Spearman r should rise monotonically with rarefaction depth, reaching r=0.82 at ~10^5 assigned reads versus r=0.18 for the shallow metagenome
- The r-versus-depth table should show the ~72,000 total-sequences-per-sample crossover below which PICRUSt outperforms metagenomic sequencing
- Depth matching must reproduce the correction for gene annotation survival (17.3%) and OTU-picking rates (68.9%) before correlating KO tables
- Per-sample correlations should be computed only after copy-number normalization of the OTU table; unnormalized OTU input should change the result
- Output CSV should contain one Spearman r per matched depth with consistent sample pairing between predicted and observed KO tables

## Limitations

- PICRUSt does not infer viral or eukaryotic metagenome components, so observed metagenomic KOs from those sources depress correlation
- Accuracy depends on availability of related reference genomes and should be interpreted alongside the per-sample NSTI value
- Gene gain/loss and laterally transferred genes will be poorly predicted, contributing broad uncertainty; rarefaction conclusions are specific to the soil dataset and its 17.3% annotation survival and 68.9% OTU-picking rates

## Evidence
