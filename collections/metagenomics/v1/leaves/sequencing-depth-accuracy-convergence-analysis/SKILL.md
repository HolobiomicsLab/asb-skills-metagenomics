---
name: sequencing-depth-accuracy-convergence-analysis
description: 'Decision support for choosing between 16S+PICRUSt prediction and shallow
  shotgun sequencing: use when paired 16S OTU tables and HUMAnN-annotated KO tables
  from the same samples are available and the question is accuracy as a function of
  sequencing depth.'
when_to_use_negative:
- The shotgun metagenomes are deeply sequenced (near or above ~72,000–10^5 reads per
  sample), where sequencing accuracy converges and PICRUSt no longer outperforms it
- The community is dominated by organisms with high NSTI (no closely related reference
  genomes), since PICRUSt accuracy depends on availability of related reference genomes
- The sample contains substantial viral or eukaryotic metagenome components, which
  PICRUSt's bacterial/archaeal marker gene framework does not infer
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_0084
tools:
- name: PICRUSt
  role: Normalizes the OTU table by 16S copy number and predicts per-sample KO metagenomes
    that are compared against rarefied observed KOs
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Builds the PICRUSt-compatible OTU table via the closed-reference OTU picking
    protocol (pick_reference_otus_through_otu_table.py) using uclust
- name: HUMAnN v0.98
  role: Annotates WGS reads to KOs to produce the observed metagenome KO abundance
    table used as the sequencing-depth ground truth
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:4317712d2e8a0f05698e74e41d93ecbc9ac936645c97f34e9a74e057bdc4521e
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
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# sequencing-depth-accuracy-convergence-analysis

## Summary

Rarefies paired 16S-derived PICRUSt metagenome predictions and shallow shotgun metagenomic KO profiles over a matched depth series and compares them by Spearman rank correlation to find the read depth at which PICRUSt outperforms sequencing. This formalizes the trade-off between prediction error and metagenome sampling depth.

## When to use

Decision support for choosing between 16S+PICRUSt prediction and shallow shotgun sequencing: use when paired 16S OTU tables and HUMAnN-annotated KO tables from the same samples are available and the question is accuracy as a function of sequencing depth.

## When NOT to use

- The shotgun metagenomes are deeply sequenced (near or above ~72,000–10^5 reads per sample), where sequencing accuracy converges and PICRUSt no longer outperforms it
- The community is dominated by organisms with high NSTI (no closely related reference genomes), since PICRUSt accuracy depends on availability of related reference genomes
- The sample contains substantial viral or eukaryotic metagenome components, which PICRUSt's bacterial/archaeal marker gene framework does not infer

## Inputs

- 16S OTU table produced by QIIME 1.5.0-dev closed-reference OTU picking (uclust vs Greengenes+IMG)
- Observed KO abundance table from HUMAnN v0.98 annotation of shotgun metagenomic reads
- PICRUSt reference data: pre-calculated predicted genome contents and 16S copy-number predictions for the marker gene tree
- List of matched rarefaction depths spanning shallow counts to 10^5 assigned reads

## Outputs

- spearman_vs_depth.csv: table of Spearman r between PICRUSt-predicted and rarefied observed KO abundances at each sequencing depth
- spearman_vs_depth.png: correlation-versus-depth figure marking the convergence point (r=0.82 at 10^5 reads) and the ~72,000-read crossover
- Rarefied 16S OTU tables and rarefied observed KO tables at each matched depth

## How to apply

Build the 16S OTU table with QIIME 1.5.0-dev closed-reference OTU picking (pick_reference_otus_through_otu_table.py with uclust against Greengenes+IMG) and annotate the shotgun metagenome KOs with HUMAnN v0.98. Run PICRUSt: normalize the OTU table by predicted 16S copy number, then multiply normalized OTU abundance by per-genome KO abundances and sum per sample to obtain predicted metagenomes. Rarefy both the 16S OTU table and the observed KO table by randomly selecting a fixed number of counts per sample across matched depths spanning shallow to 10^5 assigned reads, so that PICRUSt-predicted KOs (from 16S rarefactions) and observed KOs (from metagenome rarefactions) are compared at equivalent effort. At each depth compute the per-sample Spearman rank correlation between predicted and rarefied observed KO abundances. Interpret using the paper's calibration: at 10^5 reads the metagenome converges (r=0.82 vs r=0.18 for the shallow metagenome), and after correcting for gene annotation survival (17.3%) and OTU-picking rates (68.9%), PICRUSt outperforms subsampled metagenomes below ~72,000 total sequences per sample (with ~15,000 annotated metagenomic sequences matching PICRUSt's accuracy at 105 assigned 16S reads). Write the correlation-versus-depth table (spearman_vs_depth.csv) and figure (spearman_vs_depth.png).

## Related tools

- **PICRUSt** (Normalizes the OTU table by 16S copy number and predicts per-sample KO metagenomes that are compared against rarefied observed KOs) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Builds the PICRUSt-compatible OTU table via the closed-reference OTU picking protocol (pick_reference_otus_through_otu_table.py) using uclust)
- **HUMAnN v0.98** (Annotates WGS reads to KOs to produce the observed metagenome KO abundance table used as the sequencing-depth ground truth)

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(predicted_ko_per_sample, rarefied_observed_ko_per_sample)
```

## Evaluation signals

- At matched 10^5-read depth, the observed-metagenome vs PICRUSt Spearman r should approach the paper's convergence value (r=0.82), far above the shallow-metagenome r=0.18
- Spearman r versus depth should increase monotonically toward convergence; a non-monotonic curve suggests rarefaction or table-matching errors
- The estimated crossover depth should fall near ~72,000 total sequences per sample after applying the 17.3% gene annotation survival and 68.9% OTU-picking rate corrections
- Rarefaction must use the same fixed count per sample for both the 16S-derived and observed KO tables at each depth so the depth axis is comparable
- spearman_vs_depth.csv exists with one Spearman r per depth per sample, and spearman_vs_depth.png reproduces the correlation-versus-depth plot with the convergence and crossover marked

## Limitations

- PICRUSt does not infer viral or eukaryotic metagenome components; only bacterial and archaeal 16S marker gene sequences are included
- PICRUSt cannot distinguish strain-level variation when the marker gene sequence is identical among strains, capping achievable correlation at shallow 16S depths
- Some gene gains, losses, and laterally transferred genes will be poorly predicted, with broad confidence intervals (Supplemental Fig. 7 referenced but not provided)
- Accuracy depends on phylogenetic proximity of reference genomes (NSTI); environments with phylogenetic gaps in the reference database require additional genomes for accurate prediction

## Evidence
