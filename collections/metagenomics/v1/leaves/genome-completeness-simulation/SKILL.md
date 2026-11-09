---
name: genome-completeness-simulation
description: Use when metagenomic bins or otherwise incomplete genomes must be scored
  with trait classifiers, and you need to quantify how much balanced accuracy degrades
  as genome completeness drops before trusting predictions on partial input.
when_to_use_negative:
- The input genomes are already complete reference genomes and no assessment of robustness
  to incompleteness is needed — train and evaluate directly instead.
- No COG presence/absence feature matrix is available and traits must be inferred
  from sequences directly (use a sequence-based tool such as phenotrex predict on
  FASTA input instead of COG-profile thinning).
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PICA
  role: Framework providing the flat-file COG profile format in which the incompleteness-thinned
    profiles are written and consumed
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: libSVM-backed plug-in used to train the trait classifiers on complete genomes
    and predict traits on the simulated incomplete profiles
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: SVM implementation underlying the PICA plug-in used for the trait predictions
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:60b2fbfbb50461eb9850fe330ad78aa00dea708f30f08eba5cdc77ac056a80d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genome-completeness-simulation

## Summary

Simulates incomplete genomes by randomly removing x% of the COGs (x = 10–90, in 10% steps, 3 replicates per level) from eggNOG 4.0 presence/absence profiles, so that SVM-based phenotypic trait models can be re-evaluated on fragmentary input mimicking metagenomic bins.

## When to use

Use when metagenomic bins or otherwise incomplete genomes must be scored with trait classifiers, and you need to quantify how much balanced accuracy degrades as genome completeness drops before trusting predictions on partial input.

## When NOT to use

- The input genomes are already complete reference genomes and no assessment of robustness to incompleteness is needed — train and evaluate directly instead.
- No COG presence/absence feature matrix is available and traits must be inferred from sequences directly (use a sequence-based tool such as phenotrex predict on FASTA input instead of COG-profile thinning).

## Inputs

- eggNOG 4.0 COG presence/absence profiles for reference genomes, in PICA flat-file profile format
- Phenotypic trait labels for the complete reference genomes (e.g. gram-negative status, psychrophily, spore-forming, phototrophy)
- Trained binary libSVM-based PICA classifier per phenotypic trait, trained on complete-genome features

## Outputs

- 27 incomplete-profile files (9 removal levels x 3 replicates), named e.g. incomplete_90_1, in PICA-compatible profile format
- accuracy_vs_completeness.csv: balanced accuracy per trait per completeness level
- trait_accuracy_curves.png: sigmoidal balanced-accuracy response curves per trait (reproducing Figure 5)

## How to apply

Load the eggNOG 4.0 COG presence/absence profiles in PICA's flat-file profile format for genomes associated with the traits of interest. For each removal level x in {10, 20, ..., 90}, randomly delete x% of the total COG entries per genome profile with a fixed random seed, and repeat the procedure 3 times per level to obtain 3 independent replicates. Write each thinned profile to a PICA-compatible file named by completeness level and replicate (e.g. incomplete_90_1, incomplete_90_2, incomplete_90_3) so the PICA libSVM plug-in consumes them directly as test sets. Then predict with the trained SVM and compute balanced accuracy per trait at each completeness level, tabulating results (accuracy_vs_completeness.csv) and plotting sigmoidal accuracy response curves per trait (trait_accuracy_curves.png). The rationale is that most traits remain reliably predictable from only 60-70% complete genomes, so the simulation establishes whether predictions on real, fragmentary bins are trustworthy.

## Related tools

- **PICA** (Framework providing the flat-file COG profile format in which the incompleteness-thinned profiles are written and consumed) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (libSVM-backed plug-in used to train the trait classifiers on complete genomes and predict traits on the simulated incomplete profiles) — https://github.com/univieCUBE/PICA
- **libSVM** (SVM implementation underlying the PICA plug-in used for the trait predictions)

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Exactly 27 incomplete-profile files exist (9 removal levels x 3 replicates).
- Each file contains exactly (100 - x)% of the original per-genome COG count, and no COGs outside the original profile appear.
- Balanced accuracy reaches near-maximal values for most traits at 60-70% completeness.
- Gram-negative status remains predictable at 30-40% completeness, while psychrophily stays difficult even at 100% completeness; spore-forming and phototrophy benefit above 70% completeness, reproducing Figure 5.
- A fixed random seed makes the 3 replicates per completeness level reproducible.

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness; predictions on strongly fragmented input are not guaranteed.
- Performance for further ecologically important traits is pending availability of sufficient training data.
- Incompleteness is modeled by purely random COG removal, which may not reflect the non-random gene-loss patterns of real genome bins.
- Psychrophily remains difficult to predict even at 100% completeness, so poor accuracy there is a training-data limitation, not an incompleteness artifact.

## Evidence

- [abstract] Most traits can be reliably predicted in only 60-70% complete genomes: "Most of the traits can be reliably predicted in only 60-70% complete genomes."
- [methods] incompleteness modeled by random COG removal: [evidence span withheld — anchor status: not_found]
- [abstract] libSVM plug-in for large-scale and incomplete genomes: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [readme] phenotrex as successor of PICA: "Please checkout the new `phenotrex` package, which succeeds PICA"
