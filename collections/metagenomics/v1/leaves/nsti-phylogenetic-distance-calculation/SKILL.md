---
name: nsti-phylogenetic-distance-calculation
description: Use when predicting metagenomes from 16S data with PICRUSt and you need
  a per-sample confidence metric, or when testing whether prediction accuracy decays
  with phylogenetic distance to reference genomes.
when_to_use_negative:
- Input is shotgun metagenome data rather than 16S marker gene sequences — NSTI/PICRUSt
  confidence scoring presumes marker-gene-only samples needing predicted metagenomes
- The community includes viral or eukaryotic components — only bacterial and archaeal
  16S marker gene sequences are included, so NSTI will not characterize those fractions
- Samples' marker gene sequences are identical among strains — the metric cannot resolve
  strain-level variation, so NSTI-based confidence will not capture intra-strain functional
  differences
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: PICRUSt
  role: 'Implements the NSTI calculation: abundance-weighted phylogenetic distance
    of each OTU to its nearest sequenced reference genome, summed to one value per
    sample'
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Produces the closed-reference OTU tables (pick_reference_otus_through_otu_table.py)
    that serve as the input for per-sample NSTI computation
- name: uclust
  role: Reference OTU picking against the Greengenes+IMG database to assign community
    OTUs to sequenced references
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:ae132142a025e0a3220a3e9af9d44797e3f5f9ec59dab72db02627d3c293ec30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PICRUSt
  - QIIME 1.5.0-dev
  - uclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nsti-phylogenetic-distance-calculation

## Summary

Computes the Nearest Sequenced Taxon Index (NSTI): the abundance-weighted phylogenetic distance, on the 16S marker gene tree, between each community OTU and its nearest sequenced reference genome, summed to one NSTI value per sample. This quantifies the expected PICRUSt metagenome-prediction accuracy for that sample.

## When to use

Use when predicting metagenomes from 16S data with PICRUSt and you need a per-sample confidence metric, or when testing whether prediction accuracy decays with phylogenetic distance to reference genomes.

## When NOT to use

- Input is shotgun metagenome data rather than 16S marker gene sequences — NSTI/PICRUSt confidence scoring presumes marker-gene-only samples needing predicted metagenomes
- The community includes viral or eukaryotic components — only bacterial and archaeal 16S marker gene sequences are included, so NSTI will not characterize those fractions
- Samples' marker gene sequences are identical among strains — the metric cannot resolve strain-level variation, so NSTI-based confidence will not capture intra-strain functional differences

## Inputs

- Closed-reference-picked 16S OTU table (QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py, uclust against Greengenes+IMG)
- 16S marker gene phylogenetic tree containing tips with known gene content (complete reference genomes) and unknown tips
- Per-OTU abundance counts per sample
- Reference genome set (IMG-derived gene content, e.g. KO/COG annotations from IMG v3.5)

## Outputs

- A single NSTI value per microbial community sample (abundance-weighted nearest-sequenced-taxon distance)
- Per-sample NSTI values plus global and per-habitat NSTI-accuracy Spearman correlation coefficients (e.g. nsti_accuracy_correlation.csv)

## How to apply

Start from a closed-reference-picked 16S OTU table produced by QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py with uclust against the combined Greengenes+IMG reference. For each OTU, find its phylogenetic distance to the nearest sequenced reference genome on the 16S marker gene tree, weight that score by the OTU's abundance, and sum the weighted OTU scores into a single NSTI value per microbial community sample, as implemented in PICRUSt. Use the NSTI to stratify or gate downstream accuracy analyses — e.g. the paper restricts mammal gut accuracy analysis to samples with NSTI < 0.05 — and to interpret confidence intervals (wide intervals indicate high uncertainty). To validate the metric, correlate per-sample NSTI against PICRUSt prediction accuracy (predicted vs. sequenced metagenome KO abundances) with Spearman rank correlation, expecting a global decay of r = -0.4 (p < 0.001) and within-habitat correlations between r = -0.25 and -0.82 (p < 0.05), then write per-sample NSTI values and correlation coefficients (e.g. to nsti_accuracy_correlation.csv).

## Related tools

- **PICRUSt** (Implements the NSTI calculation: abundance-weighted phylogenetic distance of each OTU to its nearest sequenced reference genome, summed to one value per sample) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Produces the closed-reference OTU tables (pick_reference_otus_through_otu_table.py) that serve as the input for per-sample NSTI computation)
- **uclust** (Reference OTU picking against the Greengenes+IMG database to assign community OTUs to sequenced references)

## Evaluation signals

- Exactly one non-negative NSTI value is produced per sample (all abundance-weighted OTU scores summed)
- Globally, Spearman correlation between per-sample NSTI and prediction accuracy should be strongly negative (reported r = -0.4, p < 0.001)
- Within-habitat correlations should fall in the reported range r = -0.25 to -0.82 (p < 0.05); the paper's mammal gut analysis is restricted to samples with NSTI < 0.05
- Samples with the highest NSTI should correspond to those with the widest confidence intervals on predicted gene families (uncertainty signal)
- The OTU table must be closed-reference-picked against Greengenes+IMG; NSTI is undefined for de novo OTUs lacking reference-tree placement

## Limitations

- The reference database has phylogenetic gaps for some environments; additional genome sequences are needed to fill 'gaps' and allow accurate prediction (and low NSTI)
- Some gene gains, losses, or laterally transferred genes will be poorly predicted, with broad confidence intervals as a result
- PICRUSt/NSTI covers only bacterial and archaeal 16S marker gene sequences — viral and eukaryotic components are not represented
- Cannot distinguish variation at the strain level if the marker gene sequence used is identical among strains

## Evidence
