---
name: classifier-accuracy-evaluation
description: Use when a trained genotype-to-phenotype classifier must be validated
  on independent test genomes, e.g. GIDEON II held-out species or Bergey's Manual
  bacteria, to compute accuracy and macroaccuracy.
when_to_use_negative:
- The input is a feature table or phenotype predictions already produced — only compute
  metrics, do not rerun prediction
- The species overlap the GIDEON training set — accuracy on training data inflates
  performance; use held-out or independent test sets like GIDEON II or Bergey's Manual
- The goal is exploratory phenotype annotation of incomplete genomes rather than measuring
  classifier reliability
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Traitar
  role: predicts the 67 phenotype labels per genome with phypat and phypat+PGL classifiers
    that are then scored against curated labels
  repo: https://github.com/hzi-bifo/traitar
- name: Python 2.7
  role: implementation language of Traitar used for the accuracy computation scripts
  repo: https://github.com/hzi-bifo/traitar
- name: prodigal
  role: gene prediction step generating ORFs that are annotated with Pfam families
    prior to phenotype prediction on test genomes
- name: hmmer/hmmsearch
  role: annotates predicted coding sequences with Pfam families to build the protein
    family profiles scored by the classifiers
- name: Pfam 27.0 (Pfam-A.hmm)
  role: HMM models required for producing the protein family profiles on which phenotype
    accuracy is measured
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:0ec1126231de80b7a4d70da3f86877ca328c4427fcaeb3c39743406b9bcc3243
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - Pfam 27.0 (Pfam-A.hmm)
  - Python
  - Python 2.7
  - Traitar
  - hmmer/hmmsearch
  - pip
  - prodigal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# classifier-accuracy-evaluation

## Summary

Evaluates phenotype-classifier accuracy (Traitar's phypat and phypat+PGL) by comparing predicted phenotype calls over 67 traits against curated labels on held-out test sets, reporting overall accuracy and macroaccuracy per classifier.

## When to use

Use when a trained genotype-to-phenotype classifier must be validated on independent test genomes, e.g. GIDEON II held-out species or Bergey's Manual bacteria, to compute accuracy and macroaccuracy.

## When NOT to use

- The input is a feature table or phenotype predictions already produced — only compute metrics, do not rerun prediction
- The species overlap the GIDEON training set — accuracy on training data inflates performance; use held-out or independent test sets like GIDEON II or Bergey's Manual
- The goal is exploratory phenotype annotation of incomplete genomes rather than measuring classifier reliability

## Inputs

- Protein family (Pfam 27.0) profiles of held-out bacterial genome FASTAs (296 Bergey's Manual species or 42 GIDEON II species)
- Curated phenotype labels for the test species (GIDEON microbiology section or Bergey's Manual extracts)
- Trained phypat and phypat+PGL phenotype classification models

## Outputs

- Per-classifier overall accuracy (e.g. gideon2_accuracy.tsv)
- Per-classifier macroaccuracy averaged over the 67 phenotypes
- Per-phenotype accuracy comparison between phypat and phypat+PGL

## How to apply

Run Traitar's phenotype prediction with both the phypat (phyletic-pattern-only) and phypat+PGL (phyletic pattern plus maximum-likelihood-inferred ancestral protein family gains/losses) classifiers on the protein family (Pfam) profiles of the held-out genomes. Compare predicted phenotype labels across the 67 traits against curated reference labels (GIDEON for the 42 GIDEON II species, Bergey's Manual for the 296 non-GIDEON species). Compute overall sample-level accuracy and macroaccuracy (average over phenotypes) per classifier, e.g. writing both metrics per model to gideon2_accuracy.tsv. Judge reproduction against published values within ~0.1 percentage points: on GIDEON II, macroaccuracy 85.3% (phypat) and 86.7% (phypat+PGL) and accuracies 87.5% and 87.9%; on Bergey's Manual, accuracies of 72.9% (phypat) and 72.1% (phypat+PGL). Accept a run when phypat+PGL matches or improves phypat per-phenotype and overall.

## Related tools

- **Traitar** (predicts the 67 phenotype labels per genome with phypat and phypat+PGL classifiers that are then scored against curated labels) — https://github.com/hzi-bifo/traitar
- **Python 2.7** (implementation language of Traitar used for the accuracy computation scripts) — https://github.com/hzi-bifo/traitar
- **prodigal** (gene prediction step generating ORFs that are annotated with Pfam families prior to phenotype prediction on test genomes)
- **hmmer/hmmsearch** (annotates predicted coding sequences with Pfam families to build the protein family profiles scored by the classifiers)
- **Pfam 27.0 (Pfam-A.hmm)** (HMM models required for producing the protein family profiles on which phenotype accuracy is measured)

## Examples

```
traitar phenotype <in dir> <sample file> from_nucleotides <out_dir>
```

## Evaluation signals

- GIDEON II macroaccuracy reproduces 85.3% for phypat and 86.7% for phypat+PGL within ~0.1 percentage points for rounding
- GIDEON II overall accuracy reproduces 87.5% (phypat) and 87.9% (phypat+PGL)
- Bergey's Manual accuracy on the 296 non-GIDEON species is ~72.9% for phypat and ~72.1% for phypat+PGL
- phypat+PGL matches or exceeds phypat for individual phenotypes and overall, as reported
- Metric tables contain exactly one accuracy and one macroaccuracy per classifier across the 67 predicted phenotypes

## Limitations

- On the heterogeneous Bergey's Manual collection (mostly draft genomes), Traitar was less reliable than on GIDEON data (72.9%/72.1% vs ~87%)
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome, not a true negative
- Inference uses only presence/absence of protein families; other omics data are not integrated

## Evidence

- [discussion] phypat+PGL improves over phyletic-pattern-only prediction: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [intro] Traitar predicts 67 phenotypes from genome sequence: "the accurate prediction of currently 67 phenotypes directly from a genome sequence"
- [methods] Pfam 27.0 HMM models are a prerequisite: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] Standard Traitar workflow produces the protein family profiles scored on test sets: "which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs"
