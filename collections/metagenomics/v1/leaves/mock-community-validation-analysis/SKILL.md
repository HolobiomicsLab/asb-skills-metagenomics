---
name: mock-community-validation-analysis
description: Use when you have predicted gene-family (KO) abundance tables from 16S
  data plus shotgun-derived KO annotations for the same or mock communities, and need
  to quantify prediction accuracy.
when_to_use_negative:
- Input is already shotgun metagenome data with direct functional annotations — no
  prediction or mock-community validation is needed
- The communities of interest lack closely related reference genomes (high NSTI),
  so accuracy cannot be expected to generalize from the HMP/mock benchmarks
- You need to infer viral or eukaryotic metagenome components — PICRUSt only includes
  bacterial and archaeal marker gene sequences
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: PICRUSt
  role: Predicts KO metagenomes from normalized 16S OTU tables via ancestral state
    reconstruction; provides the pre-calculated gene-by-genome reference table
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Constructs PICRUSt-compatible OTU tables via closed-reference OTU picking
    (pick_reference_otus_through_otu_table.py)
- name: uclust
  role: OTU picking against the Greengenes+IMG reference for the 16S OTU table
- name: HUMAnN v0.98
  role: Annotates shotgun WGS reads to KOs to produce the ground-truth KO abundances
    used for correlation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:ea67ad166cdc40aa779a1dd55bc006b6124b8f71f305cc57e29a3ee5dfa7a6aa
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mock-community-validation-analysis

## Summary

Validates 16S-based metagenome predictions (e.g. PICRUSt KO tables) by comparing them to shotgun-annotated abundances via Spearman correlation, using HMP synthetic even/staggered mock communities and paired 16S/shotgun HMP samples as ground truth.

## When to use

Use when you have predicted gene-family (KO) abundance tables from 16S data plus shotgun-derived KO annotations for the same or mock communities, and need to quantify prediction accuracy.

## When NOT to use

- Input is already shotgun metagenome data with direct functional annotations — no prediction or mock-community validation is needed
- The communities of interest lack closely related reference genomes (high NSTI), so accuracy cannot be expected to generalize from the HMP/mock benchmarks
- You need to infer viral or eukaryotic metagenome components — PICRUSt only includes bacterial and archaeal marker gene sequences

## Inputs

- HMP 16S OTU table from paired 16S/shotgun samples (530 paired samples; closed-reference picked against Greengenes+IMG with uclust via QIIME 1.5.0-dev)
- HMP synthetic mock community 16S data (even and staggered)
- PICRUSt pre-calculated reference gene-by-genome table (predicted KO contents for all tips in the marker reference tree, derived from IMG annotations)
- Shotgun-annotated KO abundance tables (WGS reads annotated to KOs with HUMAnN v0.98)

## Outputs

- Predicted KO abundance table per sample
- Spearman correlation results between predicted and annotated KO abundances (e.g. written to picrust_ko_spearman_correlations.csv)

## How to apply

Build PICRUSt-compatible OTU tables using closed-reference OTU picking against Greengenes+IMG with uclust in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py), normalize by predicted 16S copy number, and run ancestral state reconstruction on the pruned marker gene tree to predict KO abundances per sample. Obtain true KO abundances by annotating shotgun reads with HUMAnN v0.98. Then compute Spearman correlations between predicted and annotated abundances across all 6,885 KO groups, both for the paired HMP sample set and separately for the even and staggered HMP synthetic mock communities. Success is judged by whether correlations match reported benchmarks: Spearman r≈0.82 (p<0.001) for HMP samples across body sites and r≈0.9 (p<0.001) for the synthetic mock communities.

## Related tools

- **PICRUSt** (Predicts KO metagenomes from normalized 16S OTU tables via ancestral state reconstruction; provides the pre-calculated gene-by-genome reference table) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Constructs PICRUSt-compatible OTU tables via closed-reference OTU picking (pick_reference_otus_through_otu_table.py))
- **uclust** (OTU picking against the Greengenes+IMG reference for the 16S OTU table)
- **HUMAnN v0.98** (Annotates shotgun WGS reads to KOs to produce the ground-truth KO abundances used for correlation)

## Evaluation signals

- Spearman r for the HMP paired-sample set across all 6,885 KOs is approximately 0.82 with p<0.001
- Spearman r for the HMP even and staggered synthetic mock communities is approximately 0.9 with p<0.001
- Correlation is computed across the full KO table (all 6,885 KO groups), not a subset of KOs
- Predicted KO abundances derive from a 16S copy-number-normalized OTU table plus ancestral state reconstruction, not raw OTU counts
- Ground-truth KO abundances come from HUMAnN v0.98 annotation of WGS reads, ensuring a like-for-like KO feature space

## Limitations

- PICRUSt accuracy depends on the availability of related reference genomes, quantified by NSTI; poorly covered environments will show lower accuracy than the HMP benchmark
- Only bacterial and archaeal marker gene sequences are included, so viral and eukaryotic metagenome components are not predicted and are excluded from validation
- Some gene gains, losses, or laterally transferred genes will be poorly predicted, with broad confidence intervals
- The software cannot distinguish strain-level variation if the marker gene sequence is identical among strains

## Evidence
