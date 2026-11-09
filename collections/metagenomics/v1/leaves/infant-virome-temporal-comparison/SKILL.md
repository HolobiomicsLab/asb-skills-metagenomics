---
name: infant-virome-temporal-comparison
description: Use when a vOTU count table from a longitudinal infant virome (e.g.,
  Liang et al. VLP metagenomes sampled at 0-4 days, 1 month, and 4 months) must be
  compared across sampling-time groups to test whether virome diversity increases
  with age.
when_to_use_negative:
- The input is raw sequencing reads rather than a vOTU count table — first run the
  MetaPhage reads-to-report pipeline (QC, assembly, phage detection, classification,
  quantification) before diversity computation.
- The dataset is not longitudinal/age-stratified (no sampling-month metadata), so
  the month-0-vs-month-1-vs-month-4 comparison cannot be defined.
- The goal is phage detection or classification itself rather than downstream diversity
  comparison of already-quantified vOTUs.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaPhage
  role: Produces the vOTU count table and the alpha-diversity report (Fig. 4B-style
    plots driven by alpha_var1/alpha_var2 metadata variables) that this comparison
    reproduces
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: vegan (R)
  role: Computes per-sample diversity indices (observed richness, Shannon, Simpson,
    Fisher's alpha) from the raw vOTU count matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:07673e0ca814096d8bc428f079d039278cd4842a568930b4ee7df736d4e127b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - MetaPhage
  - vegan (R)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# infant-virome-temporal-comparison

## Summary

Reproduces the age-stratified alpha-diversity pattern (richness and diversity of vOTUs) in the Liang et al. infant gut VLP virome dataset by computing observed richness, Shannon, Simpson, and Fisher's alpha per sample with vegan-style diversity functions and comparing month 0, month 1, and month 4 groups, mirroring the MetaPhage alpha-diversity report (Fig. 4B).

## When to use

Use when a vOTU count table from a longitudinal infant virome (e.g., Liang et al. VLP metagenomes sampled at 0-4 days, 1 month, and 4 months) must be compared across sampling-time groups to test whether virome diversity increases with age.

## When NOT to use

- The input is raw sequencing reads rather than a vOTU count table — first run the MetaPhage reads-to-report pipeline (QC, assembly, phage detection, classification, quantification) before diversity computation.
- The dataset is not longitudinal/age-stratified (no sampling-month metadata), so the month-0-vs-month-1-vs-month-4 comparison cannot be defined.
- The goal is phage detection or classification itself rather than downstream diversity comparison of already-quantified vOTUs.

## Inputs

- vOTU count table (samples x vOTUs) produced by the MetaPhage pipeline run on the infant VLP virome dataset
- metadata CSV with 'Sample' as its first column and a sampling-time-point variable encoding month 0, month 1, and month 4

## Outputs

- per-sample alpha-diversity table (observed richness, Shannon, Simpson, Fisher's alpha)
- boxplot/violin-style comparison of each diversity index grouped by sampling month (analogous to Fig. 4B)
- validation verdict that month-0 richness/diversity is lower than months 1 and 4

## How to apply

Load the vOTU count table (the MetaPhage run output, e.g., the task_001 count table) together with a metadata CSV whose first column is 'Sample', and merge them on sample identifiers. Map each sample to its sampling time point (month 0, month 1, month 4) via the metadata variable, matching MetaPhage's alpha_var1/alpha_var2 metadata-driven alpha-diversity plots. From the raw vOTU count matrix compute per-sample observed richness, Shannon index, Simpson index, and Fisher's alpha (e.g., with the vegan diversity functions in R). Aggregate the indices by month group and render boxplot/violin comparisons analogous to Fig. 4B. Validate that month 0 shows lower richness and diversity than months 1 and 4, consistent with the paper's reported concordance with Liang et al. on increased richness from month 0 to 4.

## Related tools

- **MetaPhage** (Produces the vOTU count table and the alpha-diversity report (Fig. 4B-style plots driven by alpha_var1/alpha_var2 metadata variables) that this comparison reproduces) — https://github.com/MattiaPandolfoVR/MetaPhage
- **vegan (R)** (Computes per-sample diversity indices (observed richness, Shannon, Simpson, Fisher's alpha) from the raw vOTU count matrix)

## Examples

```
Rscript -e 'library(vegan); library(ggplot2); otu <- read.csv("votu_counts.csv", row.names=1); meta <- read.csv("meta.csv"); H <- diversity(t(otu), index="shannon"); boxplot(H ~ meta$month)'
```

## Evaluation signals

- Merged table has one row per sample with a non-null month assignment for all 20 Liang et al. neonatal virome samples.
- Per-sample richness equals the count of vOTUs with nonzero abundance in that sample's column of the count matrix.
- Aggregated boxplots show the month-0 group with lower observed richness and diversity values than the month-1 and month-4 groups.
- The recovered richness trend matches the article's reported 'pattern of increased richness from month 0 to 4' and the Liang et al. findings.
- Metadata merge leaves no duplicated or dropped samples (row count unchanged after join on sample identifiers).

## Limitations

- MetaPhage authors urge users to be mindful of the biases inherent to virome analyses; diversity estimates inherit recall/precision trade-offs of upstream virus prediction.
- Diversity pattern is a validation-only comparison on the 20-sample Liang et al. demo dataset; statistical power is limited by small group sizes.
- The comparison depends on the upstream vOTU prediction step; CheckV showed vOTU counts can inflate with partial and low-quality genome predictions, which would bias richness.

## Evidence

- [discussion] High concordance with Liang et al. in vOTU counts and richness pattern from month 0 to 4: "we saw high concordance in the number of vOTUs predicted and a pattern of increased richness from month 0 to 4"
- [other] Workflow defining the month-grouped alpha-diversity computation: [evidence span withheld — anchor status: not_found]
- [methods] Article characterization of the infant validation dataset: "We will analyse the 20 samples from "*The stepwise assembly of the neonatal virome is modulated by breastfeeding*" ([Liang et al 2020]"
- [other] MetaPhage alpha-diversity report driven by metadata variables: [evidence span withheld — anchor status: not_found]
- [discussion] CheckV caveat on inflated vOTU counts: "Quality assessment with CheckV (29) showed that the large difference in number of vOTUs predicted was due to predictions of more partial and low-quality genomes"
