---
name: 16s-otu-rarefaction-depth-normalization
description: Use when benchmarking 16S-based metagenome prediction (e.g. PICRUSt KOs)
  against observed shotgun KO profiles as a function of sequencing depth, to identify
  the read depth at which prediction equals or exceeds a rarefied metagenome.
when_to_use_negative:
- You already have deep shotgun metagenomes for every sample — rarefying to shallow
  depths only tests prediction against artificially degraded data, not real sequencing
  alternatives
- The community includes substantial viral or eukaryotic components, since PICRUSt's
  reference contains only bacterial and archaeal 16S marker gene sequences and does
  not infer those components
- Samples have very high NSTI values (reference database phylogenetic gaps), so predicted
  metagenomes carry wide confidence intervals and depth-based correlation comparisons
  are dominated by prediction uncertainty
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: PICRUSt
  role: Predicts metagenomes from the OTU table by 16S copy-number normalization and
    weighted summing of per-genome KO abundances; its accuracy is the quantity being
    benchmarked across depths
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Generates the PICRUSt-compatible OTU table via the closed-reference OTU picking
    protocol that feeds the rarefaction analysis
- name: uclust
  role: Reference OTU picker used to assign 16S reads against Greengenes+IMG within
    the closed-reference protocol
- name: HUMAnN v0.98
  role: Annotates WGS reads to KOs to produce the observed metagenomic KO abundance
    table being rarefied and correlated against predictions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:241f06c2ee75aadd8b4e3287a5322a9412bbb01d9976e60dfa5ed8c616faca05
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# 16s-otu-rarefaction-depth-normalization

## Summary

Rarefaction-based benchmarking that compares PICRUSt-predicted KO metagenomes (from 16S OTU tables) against rarefied shallow shotgun metagenomes across matched read depths, using Spearman rank correlation to find the crossover depth (~72,000 total sequences) below which prediction outperforms sequencing.

## When to use

Use when benchmarking 16S-based metagenome prediction (e.g. PICRUSt KOs) against observed shotgun KO profiles as a function of sequencing depth, to identify the read depth at which prediction equals or exceeds a rarefied metagenome.

## When NOT to use

- You already have deep shotgun metagenomes for every sample — rarefying to shallow depths only tests prediction against artificially degraded data, not real sequencing alternatives
- The community includes substantial viral or eukaryotic components, since PICRUSt's reference contains only bacterial and archaeal 16S marker gene sequences and does not infer those components
- Samples have very high NSTI values (reference database phylogenetic gaps), so predicted metagenomes carry wide confidence intervals and depth-based correlation comparisons are dominated by prediction uncertainty

## Inputs

- 16S OTU table built with QIIME 1.5.0-dev closed-reference OTU picking (uclust against Greengenes+IMG)
- Shotgun metagenome KO abundance table annotated with HUMAnN v0.98
- PICRUSt 16S copy-number prediction file for the marker-gene reference tree
- Greengenes+IMG reference gene-by-genome KO table and marker gene tree

## Outputs

- spearman_vs_depth.csv — Spearman r per sequencing depth per sample
- spearman_vs_depth.png — correlation-versus-depth plot marking the 10^5-read convergence (r=0.82 vs r=0.18) and the ~72,000-read crossover
- Estimated crossover read depth (in total sequences per sample) below which PICRUSt outperforms the subsampled metagenome

## How to apply

Build the 16S OTU table with QIIME 1.5.0-dev's closed-reference OTU picking protocol (pick_reference_otus_through_otu_table.py, uclust against Greengenes+IMG) and annotate the shotgun metagenome to KOs with HUMAnN v0.98. Run PICRUSt: first normalize the OTU table by 16S copy number predictions, then predict the metagenome by multiplying the normalized OTU abundance by each KO abundance in the genome and summing per sample. Rarefy both the 16S OTU table and the observed KO table by randomly selecting a fixed number of counts per sample at a series of matched depths spanning shallow up to 10^5 assigned reads; keep the PICRUSt prediction fixed and rarefy the observed KO profile to comparable depths. At each depth compute the Spearman rank correlation between predicted and rarefied observed KO abundances per sample. Interpret results through the sequencing-effort correction: gene annotation survival (17.3%) and OTU-picking rates (68.9%) mean ~15,000 annotated metagenomic sequences match PICRUSt's accuracy from 105 assigned 16S reads, giving a crossover at ~72,000 total sequences where Spearman r rises to 0.82 at 10^5 reads versus r=0.18 for the shallow metagenome.

## Related tools

- **PICRUSt** (Predicts metagenomes from the OTU table by 16S copy-number normalization and weighted summing of per-genome KO abundances; its accuracy is the quantity being benchmarked across depths) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Generates the PICRUSt-compatible OTU table via the closed-reference OTU picking protocol that feeds the rarefaction analysis)
- **uclust** (Reference OTU picker used to assign 16S reads against Greengenes+IMG within the closed-reference protocol)
- **HUMAnN v0.98** (Annotates WGS reads to KOs to produce the observed metagenomic KO abundance table being rarefied and correlated against predictions)

## Evaluation signals

- Each rarefied KO/OTU sample sums to exactly the fixed count total chosen for that depth
- The observed-metagenome Spearman r increases monotonically with depth, converging to ~0.82 at 10^5 reads while the shallow metagenome sits near 0.18
- The estimated crossover (where the PICRUSt curve overtakes the rarefied metagenome curve) falls near 72,000 total sequences per sample given 17.3% annotation survival and 68.9% OTU-picking rate
- PICRUSt predictions are identical across depths — only the observed KO table and (optionally) the OTU table are rarefied
- spearman_vs_depth.csv contains one row per depth with finite Spearman coefficients and the figure marks the convergence and crossover points

## Limitations

- PICRUSt accuracy depends on availability of related reference genomes, quantified by NSTI; environments with phylogenetic gaps in the reference database require additional genomes for accurate prediction
- PICRUSt cannot distinguish strain-level variation when the marker gene sequence is identical among strains, and does not infer viral or eukaryotic metagenome components
- Some instances of gene gain or loss, or laterally transferred genes, will be poorly predicted, producing broad confidence intervals (Supplemental Fig. 7)

## Evidence

- [discussion] First, PICRUSt normalizes the OTU table by the 16S copy number predictions: "First, PICRUSt normalizes the OTU table by the 16S copy number predictions"
- [discussion] metagenome prediction step: "multiplying the normalized OTU abundance by each KO abundance in the genome and summing these KO abundances together per sample"
- [discussion] rarefaction design: "a given number of counts were randomly selected from either the collection of microbial OTUs for each sample"
- [discussion] OTU table construction: "PICRUSt-compatible OTU tables were constructed using the closed-reference OTU picking protocol in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py)"
- [readme] scope of PICRUSt prediction: "The primary application is to predict gene family abundance (e.g. the metagenome) in environmental DNA samples for which only marker gene (e.g. 16S rRNA gene) data are available."
