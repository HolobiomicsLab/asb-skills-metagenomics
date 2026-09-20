---
name: lefse-differential-abundance-analysis
description: Use when you have per-subject species/UMGS abundance profiles (one row
  per taxon, one column per sample with CD/UC/healthy class labels) and need to flag
  taxa differentially abundant across disease states with effect-size ranking.
when_to_use_negative:
- Input is raw metagenomic fastq reads rather than per-subject species abundance profiles
  — first run the DeepMicrobes TFRecord conversion, prediction, confidence-threshold
  filtering and profiling steps
- Data are not labeled into distinct biological classes (e.g. CD/UC/healthy), since
  the Kruskal-Wallis + LDA effect-size design of LEfSe requires class comparisons
- Analysis targets species present in standard reference databases, where conventional
  alignment-based abundance quantification suffices and DeepMicrobes' uncultured-MGS
  model is unnecessary
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: DeepMicrobes
  role: Upstream tool whose species-level LSTM classification and profile-generation
    step produce the UMGS abundance profiles that this LEfSe-style analysis tests
    for differential abundance
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: R
  role: Environment in which the Kruskal-Wallis tests and LDA effect-size estimation
    are run to compile the UMGS biomarker list
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:5d6714fb2a56bdcd57ebfc10d875d81f38e3eb5fefdd79a3602192013eb23c73
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lefse-differential-abundance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

LEfSe-style differential abundance testing (Kruskal-Wallis across disease classes plus LDA effect-size scoring, cutoff >2.0) applied to per-subject species abundance profiles to identify disease-associated uncultured metagenomic species (UMGS), e.g. UMGS368 depleted in Crohn's disease and ulcerative colitis.

## When to use

Use when you have per-subject species/UMGS abundance profiles (one row per taxon, one column per sample with CD/UC/healthy class labels) and need to flag taxa differentially abundant across disease states with effect-size ranking.

## When NOT to use

- Input is raw metagenomic fastq reads rather than per-subject species abundance profiles — first run the DeepMicrobes TFRecord conversion, prediction, confidence-threshold filtering and profiling steps
- Data are not labeled into distinct biological classes (e.g. CD/UC/healthy), since the Kruskal-Wallis + LDA effect-size design of LEfSe requires class comparisons
- Analysis targets species present in standard reference databases, where conventional alignment-based abundance quantification suffices and DeepMicrobes' uncultured-MGS model is unnecessary

## Inputs

- Per-subject UMGS/species relative abundance profiles (output of DeepMicrobes profile-generation step on 106 iHMP gut metagenomes, PRJNA398089)
- Sample-to-class metadata mapping subjects to Crohn's disease, ulcerative colitis, and healthy groups
- DeepMicrobes species abundance profiles for the IBD dataset (available in MicrobeLab/DeepMicrobes-data)

## Outputs

- lefse_umgs_biomarkers.tsv — significant UMGS biomarkers passing Kruskal-Wallis significance with LDA effect size >2.0
- Ranked list of differentially abundant uncultured species (e.g. UMGS368 depleted in CD/UC) cross-checkable against Figure 5 / Supplementary Table S14

## How to apply

Take the DeepMicrobes species abundance profiles generated from the 106 iHMP (PRJNA398089) gut metagenomes and, in R, run a LEfSe-style pipeline: first a Kruskal-Wallis test of each UMGS abundance across the CD, UC and healthy groups, then linear discriminant analysis (LDA) effect-size estimation on the significant taxa, retaining biomarkers with effect size >2.0. The rationale is that DeepMicrobes' LSTM-based classifier assigns reads to uncultured metagenomic species absent from reference databases, so abundance signals for those UMGS cannot be obtained with alignment/k-mer tools and must be tested statistically after profiling. Compile the significant UMGS list (e.g. UMGS368 depleted in both CD and UC) and cross-check it against the paper's Figure 5 / Supplementary Table S14 biomarker set. The workflow documented in the DeepMicrobes README — converting fastq to TFRecord, predicting with the pretrained species-level model, choosing the confidence threshold, then generating taxonomic profiles — supplies the upstream input to this step.

## Related tools

- **DeepMicrobes** (Upstream tool whose species-level LSTM classification and profile-generation step produce the UMGS abundance profiles that this LEfSe-style analysis tests for differential abundance) — https://github.com/MicrobeLab/DeepMicrobes
- **R** (Environment in which the Kruskal-Wallis tests and LDA effect-size estimation are run to compile the UMGS biomarker list) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Evaluation signals

- lefse_umgs_biomarkers.tsv must contain only UMGS passing Kruskal-Wallis significance with LDA effect size >2.0
- Calls must reproduce the Figure 5 / Supplementary Table S14 biomarker set, including UMGS368 depleted in both Crohn's disease and ulcerative colitis
- Every retained biomarker's abundance direction should be reported consistently across CD and UC versus healthy where the paper reports depletion (e.g. UMGS368 decreases in both diseases)
- Per-subject abundance profiles must cover all 106 iHMP samples with class labels assigned before testing

## Limitations

- The IBD signatures are described as 'potential'; some corroborate previous genus-level findings while others are novel, so corroboration status should be reported explicitly
- DeepMicrobes requires retraining the entire deep neural network to add new species, and adding uncultured species to reference databases is difficult and time-consuming, so biomarker lists depend on the trained repertoire (2,505 gut species)
- Biomarker detection depends on the DeepMicrobes confidence threshold chosen via the documented procedure, which filters species calls upstream

## Evidence

- [other] retaining biomarkers with effect size >2.0: [evidence span withheld — anchor status: not_found]
