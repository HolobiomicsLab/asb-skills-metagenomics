---
name: ibd-microbiome-biomarker-interpretation
description: Use when reanalyzing gut metagenomes (e.g. 106 iHMP / PRJNA398089 samples)
  from IBD cohorts (CD, UC, healthy) to find differentially abundant uncultured metagenomic
  species as disease biomarkers.
when_to_use_negative:
- When the target species are already in curated reference databases, where alignment/k-mer
  tools (e.g. Kraken) suffice and building a custom database is practical
- When reads have strongly different lengths or error profiles from short reads (e.g.
  long-read sequencing), since the pretrained model would require retraining
- When the goal is genus-level signatures only with established cultured taxa, rather
  than novel uncultured MGS associations
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DeepMicrobes
  role: Converts FASTQ to TFRecord, predicts species per read with the pretrained
    LSTM DNN, and generates taxonomic (species abundance) profiles for the IBD gut
    metagenome dataset
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: R
  role: Runs the LEfSe-style differential abundance analysis (Kruskal-Wallis test
    + LDA effect size) on UMGS abundance profiles to identify biomarkers
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:e4bb702b7c5e5e4975d9f1f2a5f673b678e9a1a9843e620e0c104da63b0a48f0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - DeepMicrobes
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ibd-microbiome-biomarker-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discover IBD-associated uncultured metagenomic species (UMGS/MGS) by classifying gut metagenome reads with DeepMicrobes' pretrained LSTM species model and identifying differentially abundant biomarkers with Kruskal-Wallis + LDA (LEfSe-style) analysis in R. Enables investigation of novel uncultured organisms from genome reconstruction that classical reference-database classifiers miss.

## When to use

Use when reanalyzing gut metagenomes (e.g. 106 iHMP / PRJNA398089 samples) from IBD cohorts (CD, UC, healthy) to find differentially abundant uncultured metagenomic species as disease biomarkers.

## When NOT to use

- When the target species are already in curated reference databases, where alignment/k-mer tools (e.g. Kraken) suffice and building a custom database is practical
- When reads have strongly different lengths or error profiles from short reads (e.g. long-read sequencing), since the pretrained model would require retraining
- When the goal is genus-level signatures only with established cultured taxa, rather than novel uncultured MGS associations

## Inputs

- Gut metagenome FASTQ files (e.g. 106 iHMP samples, accession PRJNA398089)
- DeepMicrobes pretrained species-level LSTM DNN model trained on 2,505 gut bacterial genomes
- Subject metadata assigning samples to Crohn's disease, ulcerative colitis, and healthy groups

## Outputs

- Species abundance profile per subject (taxonomic profiles from DeepMicrobes profile step)
- LEfSe-style biomarker table (lefse_umgs_biomarkers.tsv) listing UMGS passing Kruskal-Wallis significance with LDA effect size >2.0, including UMGS368 depleted in CD/UC

## How to apply

Download the iHMP gut metagenome FASTQ files (PRJNA398089) and convert fastq/fasta sequences to TFRecord format with DeepMicrobes' conversion utility. Run DeepMicrobes prediction on the TFRecord data with the pretrained species-level LSTM DNN (Embed + LSTM + Attention), choosing the confidence threshold per the documented procedure to filter species calls. Aggregate per-read species predictions into species abundance profiles, one per subject, with DeepMicrobes' profile-generation step. In R, run a LEfSe-style differential abundance analysis: Kruskal-Wallis test of UMGS abundances across CD, UC and healthy groups, followed by LDA effect-size estimation, retaining biomarkers with effect size >2.0. Rationale: MGS from gut-genome reconstruction are absent from reference databases, so deep learning on the complete 2,505-species bacterial repertoire of the human gut microbiota bypasses the difficult curation of a taxonomic tree; the resulting biomarker list (e.g. UMGS368 depleted in CD and UC) is cross-checked against the paper's Figure 5 / Supplementary Table S14.

## Related tools

- **DeepMicrobes** (Converts FASTQ to TFRecord, predicts species per read with the pretrained LSTM DNN, and generates taxonomic (species abundance) profiles for the IBD gut metagenome dataset) — https://github.com/MicrobeLab/DeepMicrobes
- **R** (Runs the LEfSe-style differential abundance analysis (Kruskal-Wallis test + LDA effect size) on UMGS abundance profiles to identify biomarkers) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Evaluation signals

- lefse_umgs_biomarkers.tsv contains only UMGS passing Kruskal-Wallis significance with LDA effect size >2.0
- The biomarker calls reproduce the Figure 5 / Supplementary Table S14 set, including UMGS368 whose abundance decreases in both Crohn's disease and ulcerative colitis
- Per-read species calls were filtered with a confidence threshold chosen per the documented procedure before aggregation
- Abundance profiles are aggregated at one profile per subject (not per read or per sample file) across CD, UC and healthy groups
- Predicted species are from the 2,505-genome gut repertoire used for training, with no reference-database curation step needed

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads with strongly different lengths and error profiles (e.g. long-read sequencing)
- The k-mer length used was optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (6-12 bp)
- The IBD signatures are described as potential; some corroborate previous genus-level findings while others are novel, and corroboration status is not fully resolved

## Evidence
