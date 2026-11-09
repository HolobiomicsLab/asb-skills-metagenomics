---
name: gene-presence-absence-feature-modeling
description: Use when the goal is to predict microbial phenotypic traits (e.g. gram-negative
  status, spore formation, phototrophy) from COG presence/absence profiles, especially
  for incomplete metagenomic genome bins rather than finished reference genomes.
when_to_use_negative:
- The input is already a trained classifier or a PhenDB model and the task is simply
  trait prediction on a FASTA bin — use the successor phenotrex package instead of
  re-training on COG matrices
- The trait of interest lacks sufficient labeled training genomes, which the article's
  discussion notes is still pending for further ecologically important traits
- Genomes are represented by raw sequence rather than annotated COG/eggNOG feature
  vectors and no annotation pipeline is available to derive presence/absence profiles
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PICA
  role: Framework for training and comparing phenotypic trait classifiers on COG presence/absence
    features
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Extended SVM plug-in applied to large-scale genome databases and incomplete
    genome sequences
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Underlying SVM library used by the PICA plug-in for binary trait classification
- name: phenotrex
  role: Successor package to PICA for end-to-end microbial phenotypic trait prediction
    (e.g. on metagenomic bins via PhenDB)
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:0377123ea87c55bc048b78e31e5a5094567c64900a6212684836e20252be1585
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-presence-absence-feature-modeling

## Summary

Train and apply SVM-based phenotypic trait classifiers on eggNOG 4.0 COG presence/absence feature matrices using the PICA framework's libSVM plug-in, and quantify how much balanced accuracy degrades when test genomes are only partially complete (metagenomic bins).

## When to use

Use when the goal is to predict microbial phenotypic traits (e.g. gram-negative status, spore formation, phototrophy) from COG presence/absence profiles, especially for incomplete metagenomic genome bins rather than finished reference genomes.

## When NOT to use

- The input is already a trained classifier or a PhenDB model and the task is simply trait prediction on a FASTA bin — use the successor phenotrex package instead of re-training on COG matrices
- The trait of interest lacks sufficient labeled training genomes, which the article's discussion notes is still pending for further ecologically important traits
- Genomes are represented by raw sequence rather than annotated COG/eggNOG feature vectors and no annotation pipeline is available to derive presence/absence profiles

## Inputs

- eggNOG 4.0 COG presence/absence feature matrix for complete reference genomes
- Phenotypic trait labels (binary, one trait per classifier) for the reference genomes
- Held-out test genomes for simulated-completeness evaluation

## Outputs

- accuracy_vs_completeness.csv: balanced accuracy per trait per completeness level
- trait_accuracy_curves.png: sigmoidal balanced-accuracy response curves per trait (reproducing Figure 5)
- Trained per-trait binary PICA/libSVM classifiers

## How to apply

Load the eggNOG 4.0 COG presence/absence feature matrix together with phenotypic trait labels for complete reference genomes, and train one binary libSVM-backed PICA classifier per trait. Simulate genome incompleteness by randomly removing genes from each held-out genome across a completeness gradient (near-complete down to strongly fragmented), then run the trained PICA SVM plug-in on the depleted genomes. Record balanced accuracy per trait at each completeness level into accuracy_vs_completeness.csv and plot the sigmoidal per-trait response curves as trait_accuracy_curves.png (Figure 5). The rationale is that trait-predictive signal resides in aggregate gene-family content, so most traits remain reliably predictable at only 60-70% completeness, enabling annotation of metagenomic bins.

## Related tools

- **PICA** (Framework for training and comparing phenotypic trait classifiers on COG presence/absence features) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Extended SVM plug-in applied to large-scale genome databases and incomplete genome sequences) — github.com/univieCUBE/PICA
- **libSVM** (Underlying SVM library used by the PICA plug-in for binary trait classification)
- **phenotrex** (Successor package to PICA for end-to-end microbial phenotypic trait prediction (e.g. on metagenomic bins via PhenDB)) — https://github.com/univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Curves reach near-maximal balanced accuracy for most traits at 60-70% genome completeness
- Gram-negative status remains predictable even at 30-40% completeness
- Psychrophily remains difficult to predict even at 100% completeness
- Spore-forming and phototrophy traits profit from completeness above 70%
- Reproduced Figure 5: sigmoidal balanced-accuracy vs. completeness response curves per trait

## Limitations

- Reliable trait prediction generally requires at least 60-70% genome completeness
- Performance for further ecologically important traits is pending availability of sufficient training data
- Some traits (e.g. psychrophily) remain hard to predict even with complete genomes
- PICA is superseded by the phenotrex package, which the README recommends for improved classifiers (e.g. XGBoost) and test-driven development

## Evidence

- [abstract] Extended SVM plug-in applicability: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [discussion] Completeness threshold finding: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [intro] libSVM as PICA plug-in: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] Prediction output format: [evidence span withheld — anchor status: external_doc]
