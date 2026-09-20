---
name: accuracy-vs-completeness-curve-visualization
description: Use when quantifying how genome incompleteness (e.g. metagenomic bins
  at 60-70% completeness) degrades binary phenotypic trait prediction accuracy, and
  when visualizing accuracy-vs-completeness curves per trait.
when_to_use_negative:
- Inputs are metagenomic reads or raw assemblies with no COG presence/absence annotation
  — the trait models operate on eggNOG COG features, not sequence data
- The goal is novel trait model training rather than evaluating robustness of already-trained
  trait models to genome incompleteness
- Traits lacking sufficient positive training genomes (the paper notes further ecological
  traits await sufficient training data)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PICA
  role: Framework for training binary phenotype prediction models on COG presence/absence
    features of complete reference genomes
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM classifier used both to train trait models and to predict traits on the
    simulated incomplete genomes
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Underlying SVM library included as a PICA plug-in for the binary trait classification
- name: phenotrex
  role: Successor package of PICA that can apply pre-trained trait models to genome
    files for prediction
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:b9596dec8065a8e98781173ab3b6964784a9c2c52348c5dfb7015faebcc78d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# accuracy-vs-completeness-curve-visualization

## Summary

Reproduces the paper's Figure 5 analysis: sigmoidal balanced-accuracy response curves per phenotypic trait as a function of simulated genome completeness, showing how much incompleteness SVM-based PICA trait prediction can tolerate.

## When to use

Use when quantifying how genome incompleteness (e.g. metagenomic bins at 60-70% completeness) degrades binary phenotypic trait prediction accuracy, and when visualizing accuracy-vs-completeness curves per trait.

## When NOT to use

- Inputs are metagenomic reads or raw assemblies with no COG presence/absence annotation — the trait models operate on eggNOG COG features, not sequence data
- The goal is novel trait model training rather than evaluating robustness of already-trained trait models to genome incompleteness
- Traits lacking sufficient positive training genomes (the paper notes further ecological traits await sufficient training data)

## Inputs

- eggNOG 4.0 COG presence/absence feature matrix for complete reference genomes
- Binary phenotypic trait labels per reference genome (e.g. gram-negative, psychrophily, spore-forming, phototrophy)
- Held-out complete test genomes for simulated gene removal

## Outputs

- accuracy_vs_completeness.csv (balanced accuracy per trait per completeness level)
- trait_accuracy_curves.png (sigmoidal balanced-accuracy response curves per trait, reproducing Figure 5)

## How to apply

Train one binary libSVM-based PICA classifier per phenotypic trait on the eggNOG 4.0 COG presence/absence feature matrix of complete reference genomes. Simulate incomplete test genomes by randomly removing genes from each held-out genome at a graded series of completeness levels, from near-complete down to strongly fragmented. Predict traits with the trained PICA SVM plug-in at each completeness level and compute balanced accuracy per trait per level. Tabulate the values into accuracy_vs_completeness.csv and plot sigmoidal balanced-accuracy response curves per trait (trait_accuracy_curves.png). The rationale is that SVM feature spaces built on COG presence/absence tolerate substantial gene loss, so curves reveal the completeness threshold below which each trait becomes unreliable.

## Related tools

- **PICA** (Framework for training binary phenotype prediction models on COG presence/absence features of complete reference genomes) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM classifier used both to train trait models and to predict traits on the simulated incomplete genomes) — https://github.com/univieCUBE/PICA
- **libSVM** (Underlying SVM library included as a PICA plug-in for the binary trait classification)
- **phenotrex** (Successor package of PICA that can apply pre-trained trait models to genome files for prediction) — https://github.com/univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Curves reach near-maximal balanced accuracy for most traits at 60-70% completeness
- Gram-negative status remains predictable down to 30-40% completeness
- Psychrophily stays difficult to predict even at 100% completeness
- Spore-forming and phototrophy accuracy increases above 70% completeness
- Resulting curves reproduce the qualitative shape of Figure 5 (sigmoidal response per trait)

## Limitations

- Reliable trait prediction generally requires at least 60-70% genome completeness; strongly fragmented bins are unreliable for most traits
- Some traits (e.g. psychrophily) perform poorly even on complete genomes, so low curves do not necessarily indicate incompleteness effects
- PICA is superseded by the phenotrex package; phenotrex outputs a confidence value ranging from 0.5 to 1.0 rather than PICA's raw plug-in behavior

## Evidence

- [abstract] Most traits reliably predicted in incomplete genomes: "Most of the traits can be reliably predicted in only 60-70% complete genomes."
- [abstract] SVM plug-in applicability to incomplete sequences: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [discussion] Automatic annotation of near-complete genomes: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] phenotrex confidence output semantics: [evidence span withheld — anchor status: external_doc]
