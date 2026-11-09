---
name: nested-cross-validation-model-evaluation
description: Use when benchmarking Traitar phenotype classifiers on labeled bacterial
  genomes (e.g. GIDEON phenotypes) and you need unbiased accuracy estimates such as
  macroaccuracy over 67 traits via 10-fold nested cross-validation.
when_to_use_negative:
- The goal is phenotyping new/independent genomes rather than estimating classifier
  performance on a labeled benchmark (use the standard `traitar phenotype` workflow
  instead).
- Input data lack phenotype labels, so no supervised cross-validation accuracy estimate
  can be computed.
- Only incomplete genomes or SAGs are available — the paper notes predictions on incomplete
  genomes carry more false positives and that absence of a prediction may simply reflect
  missing protein families.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: Traitar
  role: Trains the phypat and phypat+PGL phenotype classifiers and runs the nested
    cross-validation over the 67 GIDEON phenotypes
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Recommended installation route for traitar from the Python packaging index
- name: prodigal
  role: Open reading frame prediction feeding the Pfam annotation step of the Traitar
    workflow
- name: hmmsearch
  role: HMMer tool used to annotate coding sequences with Pfam families for the phyletic-pattern
    features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:f079d8d009066bcb981997d384147406f27adc2d99e6145355ad6595f904b3d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - Python
  - Traitar
  - hmmsearch
  - matplotlib
  - pip
  - prodigal
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nested-cross-validation-model-evaluation

## Summary

Reproduces Traitar's 10-fold nested cross-validation benchmark of the phypat (phyletic-pattern) and phypat+PGL (phyletic pattern plus maximum-likelihood ancestral protein family gains/losses) classifiers for 67 phenotypes across 234 GIDEON I bacterial species, reporting overall accuracy and macroaccuracy.

## When to use

Use when benchmarking Traitar phenotype classifiers on labeled bacterial genomes (e.g. GIDEON phenotypes) and you need unbiased accuracy estimates such as macroaccuracy over 67 traits via 10-fold nested cross-validation.

## When NOT to use

- The goal is phenotyping new/independent genomes rather than estimating classifier performance on a labeled benchmark (use the standard `traitar phenotype` workflow instead).
- Input data lack phenotype labels, so no supervised cross-validation accuracy estimate can be computed.
- Only incomplete genomes or SAGs are available — the paper notes predictions on incomplete genomes carry more false positives and that absence of a prediction may simply reflect missing protein families.

## Inputs

- 234 GIDEON I bacterial genome assemblies (nucleotide FASTA)
- GIDEON phenotype labels for 67 phenotypes (e.g. traits.tsv-style phenotype matrix)
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded via `traitar pfam` or supplied locally

## Outputs

- Pfam protein family presence/absence profiles per genome
- Per-classifier cross-validation metrics: overall accuracy and macroaccuracy over the 67 phenotypes (phypat and phypat+PGL)
- Metrics table written to cross_validation_accuracy.tsv

## How to apply

Install traitar from the Python packaging index with pip, install its dependencies (scipy, matplotlib, pandas, prodigal, hmmsearch, GNU parallel), and fetch the Pfam 27.0 HMM models via `traitar pfam <download_folder>` (or register a local Pfam folder with `traitar pfam --local`). Run the Traitar annotation workflow on the 234 GIDEON I genomes to obtain Pfam protein family presence/absence profiles. Train both classifiers for the 67 phenotypes: phypat uses phyletic patterns alone, while phypat+PGL additionally incorporates ancestral protein family gains and losses inferred by maximum-likelihood inference of ancestral character states on the species phylogeny, which accounts for horizontal gene transfer. Perform 10-fold nested cross-validation and compute overall accuracy and macroaccuracy per classifier across the 67 phenotypes. Accept the run if macroaccuracy reaches 82.6% (phypat) and 85.5% (phypat+PGL), with overall accuracy of 88.1% and 89.8% respectively, within a small tolerance, and write the metrics table to cross_validation_accuracy.tsv.

## Related tools

- **Traitar** (Trains the phypat and phypat+PGL phenotype classifiers and runs the nested cross-validation over the 67 GIDEON phenotypes) — https://github.com/hzi-bifo/traitar
- **pip** (Recommended installation route for traitar from the Python packaging index)
- **prodigal** (Open reading frame prediction feeding the Pfam annotation step of the Traitar workflow)
- **hmmsearch** (HMMer tool used to annotate coding sequences with Pfam families for the phyletic-pattern features)

## Examples

```
pip install traitar && traitar pfam pfam27 && traitar phenotype gideon_genomes/ samples.txt from_nucleotides traitar_out -c 4
```

## Evaluation signals

- Macroaccuracy over the 67 GIDEON phenotypes matches 82.6% for phypat and 85.5% for phypat+PGL within a small tolerance
- Overall accuracy matches 88.1% (phypat) and 89.8% (phypat+PGL) within a small tolerance
- Metrics table is produced as cross_validation_accuracy.tsv with per-classifier values
- Both classifiers are evaluated on identical 10-fold nested cross-validation splits over the 234 GIDEON I species
- phypat+PGL accuracy exceeds phypat accuracy, consistent with the paper's finding that ancestral gains/losses improve prediction

## Limitations

- Genotype-phenotype inference with Traitar only takes into account presence/absence of protein families; other omics data are not integrated.
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome.
- The plant biomass predictor prototype is not included in the released Traitar software.
- Docker usage has a problem with parallel execution, so the -c option is not guaranteed to work in the container.

## Evidence

- [results] In the 10-fold nested cross-validation on GIDEON I, the macroaccuracy over the 67 GIDEON phenotypes was 82.6% for the phypat classifier and 85.5% for the phypat+PGL classifier.: [evidence span withheld — anchor status: not_found]
- [intro] We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events: "We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events"
- [intro] The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny: "The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny"
- [methods] We strongly encourage you to install traitar with pip rather than by manually cloning the repository.: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [discussion] At the moment, genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families of the bacteria analyzed.: "At the moment, genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families of the bacteria analyzed."
