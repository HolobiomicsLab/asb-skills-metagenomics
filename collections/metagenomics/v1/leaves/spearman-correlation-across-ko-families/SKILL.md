---
name: spearman-correlation-across-ko-families
description: Use when you have paired 16S amplicon and shotgun metagenome samples
  and need to quantify agreement between predicted (PICRUSt) and directly annotated
  KO abundance profiles.
when_to_use_negative:
- Input is already shotgun metagenome data — KO abundances can be annotated directly
  (e.g. HUMAnN) rather than predicted from 16S and correlated
- Communities with substantial viral or eukaryotic components — PICRUSt only includes
  bacterial and archaeal 16S marker gene sequences and does not infer these components
- Questions requiring strain-level resolution — PICRUSt cannot distinguish variation
  at the strain level if the marker gene sequence is identical among strains
tools:
- name: PICRUSt
  role: Predicts metagenome KO abundances from normalized 16S OTU tables via ancestral
    state reconstruction; correlations are computed against its predictions
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Produces the PICRUSt-compatible closed-reference OTU table that is the prediction
    input
- name: uclust
  role: OTU picking engine for closed-reference picking against Greengenes+IMG
- name: HUMAnN v0.98
  role: Annotates WGS reads to KOs to generate the observed (shotgun) KO abundances
    used as the correlation reference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:766ee594d923f3e30d25e99fc8ecdccae58a5dd015b5631429f27269b46db5ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Greengenes
  - HUMAnN
  - HUMAnN v0.98
  - PICRUSt
  - QIIME 1.5.0-dev
  - uclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# spearman-correlation-across-ko-families

## Summary

Validates PICRUSt 16S-predicted metagenomes by computing Spearman correlations between predicted and shotgun-annotated KO (KEGG Ortholog) abundances across all 6,885 KO groups, using HMP paired 16S/shotgun samples and synthetic mock communities as benchmarks.

## When to use

Use when you have paired 16S amplicon and shotgun metagenome samples and need to quantify agreement between predicted (PICRUSt) and directly annotated KO abundance profiles.

## When NOT to use

- Input is already shotgun metagenome data — KO abundances can be annotated directly (e.g. HUMAnN) rather than predicted from 16S and correlated
- Communities with substantial viral or eukaryotic components — PICRUSt only includes bacterial and archaeal 16S marker gene sequences and does not infer these components
- Questions requiring strain-level resolution — PICRUSt cannot distinguish variation at the strain level if the marker gene sequence is identical among strains

## Inputs

- 16S OTU table (BIOM) from 530 paired HMP samples, closed-reference picked against Greengenes+IMG with uclust (QIIME 1.5.0-dev)
- PICRUSt pre-calculated reference gene-by-genome table of predicted KO contents for all tips in the marker reference tree
- Shotgun-annotated KO abundance table for paired samples (WGS reads annotated to KOs using HUMAnN v0.98)
- HMP even/staggered synthetic mock community profiles

## Outputs

- Predicted KO abundance table per sample across 6,885 KO groups
- picrust_ko_spearman_correlations.csv containing Spearman r and p-values per sample set (HMP set and each mock community)

## How to apply

Obtain the HMP 16S OTU table (closed-reference picked against Greengenes+IMG with uclust via QIIME 1.5.0-dev) and the matching shotgun-annotated KO abundances produced with HUMAnN v0.98. Run the PICRUSt pipeline: normalize the OTU table by predicted 16S copy number, then apply ancestral state reconstruction on the pruned marker gene tree and compute predicted KO abundances by multiplying normalized OTU abundance by each KO abundance in the genome and summing per sample. For each paired sample (or the pooled HMP set and each synthetic mock community), compute the Spearman correlation between predicted and annotated abundances across all 6,885 KO groups, since the correlation is rank-based and robust to the wide dynamic range of gene-family counts. Write the per-set correlation coefficients and p-values to picrust_ko_spearman_correlations.csv. Success is judged by reproducing the published values: Spearman r approximately 0.82 (p<0.001) for the HMP sample set and approximately 0.9 (p<0.001) for the HMP even/staggered synthetic mock communities.

## Related tools

- **PICRUSt** (Predicts metagenome KO abundances from normalized 16S OTU tables via ancestral state reconstruction; correlations are computed against its predictions) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Produces the PICRUSt-compatible closed-reference OTU table that is the prediction input)
- **uclust** (OTU picking engine for closed-reference picking against Greengenes+IMG)
- **HUMAnN v0.98** (Annotates WGS reads to KOs to generate the observed (shotgun) KO abundances used as the correlation reference)

## Evaluation signals

- Spearman r for the HMP sample set across all 6,885 KO groups is approximately 0.82 with p<0.001
- Spearman r for the two HMP even/staggered synthetic mock communities is approximately 0.9 with p<0.001
- Correlation vectors for predicted and annotated profiles have identical length (6,885 KO groups) and matching KO identifiers
- Results table picrust_ko_spearman_correlations.csv is populated for every body site and both mock communities
- OTU table was normalized by 16S copy number before prediction (skipping this step is a common source of depressed correlations)

## Limitations

- Accuracy depends on availability of related reference genomes, quantified by NSTI; samples with high NSTI will have less reliable predictions
- Some instances of gene gain or loss or laterally transferred genes will be poorly predicted, with broad confidence intervals
- Only bacterial and archaeal 16S marker gene sequences are included, so viral and eukaryotic metagenome components are not inferred
- No persistent identifier/accession is given for the HMP 16S data, only the URL http://hmpdacc.org/HMQCP

## Evidence
