---
name: species-abundance-profiling
description: Use when you need species-level abundance profiles from shotgun metagenome
  FASTQ data that include uncultured metagenomic species (MGS/UMGS), e.g. for IBD
  case-control biomarker discovery.
when_to_use_negative:
- Input is already a species abundance/feature table — only the downstream LEfSe step
  is needed, not read-level classification
- Reads have lengths and error profiles strongly different from short reads (e.g.
  long-read sequencing) — retraining would be required
- You need to add new species to the database without reclassification overhead —
  adding new species requires retraining the entire deep neural network
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: DeepMicrobes
  role: Converts FASTQ to TFRecord, performs per-read species-level LSTM classification
    with confidence-threshold filtering, and generates taxonomic/abundance profiles
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: R
  role: Runs LEfSe-style differential abundance analysis (Kruskal-Wallis + LDA effect
    size) on the species profiles to call UMGS biomarkers
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:0248f3c989f5c8cb5283d51e6c0da58afed5b38f6cb76e385142db78318ecece
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# species-abundance-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A deep-learning-based pipeline that converts gut metagenome FASTQ reads into TFRecords, classifies them per-read at species level (including uncultured MGS) with DeepMicrobes' pretrained LSTM model, and aggregates the calls into per-subject species abundance profiles for downstream differential-abundance testing.

## When to use

Use when you need species-level abundance profiles from shotgun metagenome FASTQ data that include uncultured metagenomic species (MGS/UMGS), e.g. for IBD case-control biomarker discovery.

## When NOT to use

- Input is already a species abundance/feature table — only the downstream LEfSe step is needed, not read-level classification
- Reads have lengths and error profiles strongly different from short reads (e.g. long-read sequencing) — retraining would be required
- You need to add new species to the database without reclassification overhead — adding new species requires retraining the entire deep neural network

## Inputs

- Shotgun gut metagenome FASTQ/FASTA files (e.g. 106 iHMP PRJNA398089 samples)
- Pretrained DeepMicrobes species-level LSTM DNN model (trained on the 2,505-species complete bacterial repertoire of the human gut microbiota)

## Outputs

- TFRecord-converted sequence files
- Per-read species-level classification calls (filtered by confidence threshold)
- Species abundance profile, one per subject (e.g. species profiles of the IBD dataset)
- lefsf-style UMGS biomarker list (e.g. lefse_umgs_biomarkers.tsv with LDA effect size >2.0)

## How to apply

Download the 106 iHMP (PRJNA398089) gut metagenome FASTQ files and convert the fastq/fasta sequences to TFRecord format with DeepMicrobes' conversion utility. Run DeepMicrobes prediction on the TFRecords with the pretrained species-level LSTM DNN model, applying the documented procedure for choosing the confidence threshold to filter species calls. Aggregate per-read species predictions into per-subject species abundance profiles with DeepMicrobes' profile-generation step. The profiles (e.g. species profiles of the IBD dataset released in DeepMicrobes-data) then feed Kruskal-Wallis + LDA effect-size (LEfSe-style) analysis in R, retaining biomarkers with effect size >2.0 such as UMGS368 depleted in CD/UC.

## Related tools

- **DeepMicrobes** (Converts FASTQ to TFRecord, performs per-read species-level LSTM classification with confidence-threshold filtering, and generates taxonomic/abundance profiles) — https://github.com/MicrobeLab/DeepMicrobes
- **R** (Runs LEfSe-style differential abundance analysis (Kruskal-Wallis + LDA effect size) on the species profiles to call UMGS biomarkers) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
python deepmicrobes/convert_to_tfrecord.py -i sample_1.fastq,sample_2.fastq -o sample.tfrecord && python deepmicrobes/predict.py -i sample.tfrecord -m deepmicrobes_models/species -t 0.8 && python deepmicrobes/profile.py -o sample_profile.tsv
```

## Evaluation signals

- The output lefse_umgs_biomarkers.tsv must contain only UMGS passing Kruskal-Wallis significance with LDA effect size >2.0
- Biomarker calls must reproduce the Figure 5 / Supplementary Table S14 set, including UMGS368 depleted in both Crohn's disease and ulcerative colitis
- Confidence threshold must be chosen via the documented procedure before accepting species calls
- Species profiles must cover the same per-subject aggregation as the IBD dataset profiles released in DeepMicrobes-data

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- The k-mer length used is optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (e.g. 6-12 bp)

## Evidence
