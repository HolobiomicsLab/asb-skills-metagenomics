---
name: ancestral-character-state-inference
description: Use when predicting bacterial phenotypes from Pfam presence/absence profiles
  and you want to incorporate horizontal gene transfer by augmenting phyletic patterns
  with ancestral gains/losses inferred on the species tree.
when_to_use_negative:
- Input is already a trained model or a phenotype prediction table — ancestral-state
  inference is a training-time feature-construction step, not needed for applying
  phypat+PGL to new genomes.
- No species phylogeny is available or the genome set is too small for meaningful
  ancestral reconstruction — fall back to the phypat phyletic-pattern-only classifier.
- The goal is exploratory phenotyping of incomplete genomes, where the article notes
  phypat assigns more phenotypes (at the price of more false positives) and may be
  preferable.
edam_operation: http://edamontology.org/operation_0550
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: Traitar
  role: Implements the phypat+PGL classifier that uses maximum-likelihood-inferred
    ancestral protein family gains/losses on the species phylogeny as additional features
    for phenotype prediction.
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Recommended installation method for traitar and its Python dependencies (scipy,
    matplotlib, pandas).
- name: prodigal
  role: Open reading frame prediction producing coding sequences that are subsequently
    annotated with Pfam families.
- name: hmmsearch
  role: HMMer tool used to annotate predicted proteins against Pfam 27.0 HMM models
    to build the phyletic patterns.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:169007247005fd487912375c5e4cbbd42eb03e932e8695c3b70ce00456827d2b
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
  edam_operation: http://edamontology.org/operation_0550
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ancestral-character-state-inference

## Summary

Maximum-likelihood inference of ancestral protein family gains and losses on a species phylogeny, used by Traitar's phypat+PGL classifier to account for horizontal gene transfer when predicting bacterial phenotypes.

## When to use

Use when predicting bacterial phenotypes from Pfam presence/absence profiles and you want to incorporate horizontal gene transfer by augmenting phyletic patterns with ancestral gains/losses inferred on the species tree.

## When NOT to use

- Input is already a trained model or a phenotype prediction table — ancestral-state inference is a training-time feature-construction step, not needed for applying phypat+PGL to new genomes.
- No species phylogeny is available or the genome set is too small for meaningful ancestral reconstruction — fall back to the phypat phyletic-pattern-only classifier.
- The goal is exploratory phenotyping of incomplete genomes, where the article notes phypat assigns more phenotypes (at the price of more false positives) and may be preferable.

## Inputs

- Pfam protein family presence/absence matrix (phyletic patterns) for bacterial genomes
- Species phylogeny of the analyzed bacteria
- Phenotype labels for the 67 GIDEON traits (e.g., from the GIDEON microbiology section)

## Outputs

- Ancestral protein family gains and losses per branch of the species tree
- Augmented feature matrix (phyletic patterns + PGL) used for training phypat+PGL models
- Phenotype predictions and cross-validation accuracy metrics (e.g., cross_validation_accuracy.tsv)

## How to apply

First generate Pfam protein family presence/absence profiles for the genomes (e.g., the 234 GIDEON I bacterial species) with Prodigal gene calling and HMMer annotation against Pfam 27.0 HMM models. Then perform maximum-likelihood inference of ancestral character states for each protein family on the species phylogeny to determine ancestral gains and losses. Concatenate these phylogeny-derived features (PGL) with the raw phyletic patterns and train the traitar phenotype classifiers for the 67 GIDEON phenotypes, comparing phypat (patterns only) against phypat+PGL. The rationale is that horizontal gene transfer means a trait's distribution across a phylogeny carries signal beyond pure presence/absence; the paper showed phypat+PGL raises macroaccuracy from 82.6% to 85.5% in 10-fold nested cross-validation.

## Related tools

- **Traitar** (Implements the phypat+PGL classifier that uses maximum-likelihood-inferred ancestral protein family gains/losses on the species phylogeny as additional features for phenotype prediction.) — https://github.com/hzi-bifo/traitar
- **pip** (Recommended installation method for traitar and its Python dependencies (scipy, matplotlib, pandas).)
- **prodigal** (Open reading frame prediction producing coding sequences that are subsequently annotated with Pfam families.)
- **hmmsearch** (HMMer tool used to annotate predicted proteins against Pfam 27.0 HMM models to build the phyletic patterns.)

## Examples

```
pip install traitar && traitar pfam <download_folder> && traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir>
```

## Evaluation signals

- In 10-fold nested cross-validation on GIDEON I (234 species, 67 phenotypes), macroaccuracy should reach ~85.5% for phypat+PGL vs ~82.6% for phypat; overall accuracy ~89.8% vs ~88.1%.
- The phypat+PGL classifier should improve accuracy over phypat 'both for individual phenotypes and overall' — a systematic drop below phypat indicates miscomputed ancestral states.
- The traitar annotation workflow must complete gene prediction (gene_prediction), Pfam annotation (pfam_annotation), and phenotype prediction (phenotype prediction) output directories without errors.
- Metrics table cross_validation_accuracy.tsv should match reported values within a small tolerance.

## Limitations

- Genotype-phenotype inference with Traitar only takes into account presence and absence of protein families; other omics data are not integrated.
- For incomplete genomes, the absence of a phenotype prediction may simply reflect missing protein families in the input genome.
- The plant biomass predictor prototype is not included in the released Traitar software.
- Phylogeny-aware prediction requires a species phylogeny; horizontal gene transfer is handled only via maximum-likelihood ancestral gains/losses, not by other mechanisms.

## Evidence

- [intro] The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny: "The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny"
- [intro] We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events: "We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events"
- [discussion] improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall.: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir>: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir>"
