---
name: metagenome-functional-prediction-from-16s
description: Use when the input is a closed-reference-picked 16S OTU table (e.g.,
  Greengenes+IMG, QIIME 1.5.0-dev/uclust) and the goal is to infer KO/gene family
  abundances or pathway-level functional profiles without shotgun metagenome data.
when_to_use_negative:
- When viral or eukaryotic metagenome components are of interest — the 16S-based system
  only covers bacterial and archaeal genomes.
- When strain-level variation is the target — the software cannot distinguish variation
  among strains with identical marker gene sequences.
- When the expected NSTI is high (broad phylogenetic gaps in the reference database),
  prediction of laterally transferred or rapidly evolving gene families will be unreliable.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt
  role: 'Core engine: copy-number normalization, ancestral-state reconstruction, and
    metagenome (KO abundance) prediction'
  repo: https://github.com/picrust/picrust
- name: QIIME 1.5.0-dev
  role: Constructs PICRUSt-compatible OTU tables via closed-reference OTU picking
    (pick_reference_otus_through_otu_table.py)
- name: uclust
  role: OTU picking/search engine used for closed-reference assignment against Greengenes+IMG
- name: HUMAnN v0.98
  role: Annotates shotgun WGS reads to KOs to generate observed abundances for validating
    PICRUSt predictions
- name: STAMP v2.0
  role: Collapses/compares predicted KEGG pathway abundances and computes Spearman
    correlations against metadata gradients
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:763fa2229ef7df0c3564e1c760697b6a2a800982dcbc651da12fc7bd56323fdb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust
  tools:
  - Greengenes
  - HUMAnN
  - HUMAnN v0.98
  - PICRUSt
  - QIIME 1.5.0-dev
  - STAMP
  - STAMP v2.0
  - uclust
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# metagenome-functional-prediction-from-16s

## Summary

Predict metagenome gene family (KO) abundance from 16S rRNA data using PICRUSt, which normalizes OTU tables by predicted 16S copy number, applies extended ancestral-state reconstruction on a marker gene tree, and combines predicted gene contents into a composite metagenome per sample.

## When to use

Use when the input is a closed-reference-picked 16S OTU table (e.g., Greengenes+IMG, QIIME 1.5.0-dev/uclust) and the goal is to infer KO/gene family abundances or pathway-level functional profiles without shotgun metagenome data.

## When NOT to use

- When viral or eukaryotic metagenome components are of interest — the 16S-based system only covers bacterial and archaeal genomes.
- When strain-level variation is the target — the software cannot distinguish variation among strains with identical marker gene sequences.
- When the expected NSTI is high (broad phylogenetic gaps in the reference database), prediction of laterally transferred or rapidly evolving gene families will be unreliable.

## Inputs

- Closed-reference-picked 16S OTU table (QIIME-compatible, picked against Greengenes+IMG with uclust)
- Pre-calculated PICRUSt reference gene-by-genome table (predicted KO/COG contents for all tips in the marker reference tree, derived from IMG v3.5 annotations)
- Predicted 16S copy number table per OTU
- Marker gene reference tree (16S phylogeny including tips with and without reference genomes)

## Outputs

- Per-sample predicted KO (or COG) abundance table (e.g., .biom)
- Per-sample weighted NSTI confidence values
- KEGG pathway-level collapsed abundance table (e.g., coral_mucus_predicted_metagenomes.biom)
- Spearman correlation summary of predicted vs. observed KO abundances (e.g., picrust_ko_spearman_correlations.csv)

## How to apply

First construct a PICRUSt-compatible OTU table by closed-reference OTU picking against Greengenes+IMG using QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py with uclust). Normalize the OTU table by predicted 16S copy number per OTU, since copy-number variation otherwise biases downstream gene abundances. Run PICRUSt metagenome prediction: ancestral state reconstruction is applied to the pruned marker gene tree (Wagner Parsimony via COUNT, ACE ML/REML/PIC via APE), gene content for tips lacking reference genomes is an average of extant and inferred ancestral genomes weighted exponentially by the reciprocal of phylogenetic distance, and the composite metagenome is produced by multiplying normalized OTU abundance by each KO abundance in the genome and summing per sample. Record the per-sample weighted NSTI as an expected-accuracy confidence score. Validate by computing Spearman correlations of predicted versus HUMAnN-annotated KO abundances (e.g., across 6,885 KO groups) and, for pathway-level questions, collapsing predicted KOs into KEGG pathways and testing associations in STAMP v2.0.

## Related tools

- **PICRUSt** (Core engine: copy-number normalization, ancestral-state reconstruction, and metagenome (KO abundance) prediction) — https://github.com/picrust/picrust
- **QIIME 1.5.0-dev** (Constructs PICRUSt-compatible OTU tables via closed-reference OTU picking (pick_reference_otus_through_otu_table.py))
- **uclust** (OTU picking/search engine used for closed-reference assignment against Greengenes+IMG)
- **HUMAnN v0.98** (Annotates shotgun WGS reads to KOs to generate observed abundances for validating PICRUSt predictions)
- **STAMP v2.0** (Collapses/compares predicted KEGG pathway abundances and computes Spearman correlations against metadata gradients)

## Evaluation signals

- Spearman correlation between predicted and observed KO abundances across all 6,885 KO groups should be high (r≈0.82, p<0.001 for the 6,431-sample HMP set).
- For HMP even/staggered synthetic mock communities, predicted vs. observed Spearman r should approach ~0.9 (p<0.001).
- Per-sample weighted NSTI should be reported and used as the expected-accuracy confidence metric (e.g., ~0.12 ± 0.02 s.d. for the coral mucus dataset).
- Pathway-level validation: reproduced Spearman correlations should match reported values (e.g., secretion systems r=1.0, ribosomal biogenesis r=0.90, galactose metabolism r=-1.0, ascorbate/alderate metabolism r=-0.90 for coral mucus vs. algal cover).
- Genome holdout validation: predicted gene content of each IMG genome, excluded from the reference gene-by-genome table, should be compared against its true annotated contents.

## Limitations

- Does not infer viral or eukaryotic metagenome components since only bacterial and archaeal 16S marker sequences are included.
- Cannot distinguish strain-level variation when the marker gene sequence is identical among strains.
- Some gene gain/loss events and laterally transferred genes will be poorly predicted, yielding broad confidence intervals (Supplemental Fig. 7).
- Accuracy depends on phylogenetic proximity of available reference genomes; environments with phylogenetic gaps in the reference database require additional genomes for accurate prediction.

## Evidence
