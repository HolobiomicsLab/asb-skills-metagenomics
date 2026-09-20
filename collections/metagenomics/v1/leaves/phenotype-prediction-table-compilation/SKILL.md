---
name: phenotype-prediction-table-compilation
description: Use when a trained phenotypic trait model exists and you need to score
  every genome/species in a large database (e.g. eggNOG 4.0 core and periphery) and
  assemble the positives into a validated prediction table.
when_to_use_negative:
- The input is a raw genome FASTA rather than precomputed COG presence/absence trait
  profiles — train/sample extraction must happen first.
- You need predictions on incomplete metagenomic genome bins with no trait model trained
  on marker COGs; the paper cautions that reliable prediction requires roughly 60-70%
  genome completeness.
- The target trait has no pre-trained model; the paper notes performance for further
  ecologically important traits awaits sufficient training data.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework that applies the trained phenotype model to score every prokaryotic
    species in the eggNOG 4.0 trait-profile matrix
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Plug-in that hosts the trained obligate intracellular SVM used to assign positive/negative
    labels to each species
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend SVM implementation behind the PICA plug-in that scores species against
    the intracellular trait model
- name: phenotrex
  role: Successor package to PICA that can run the same predict step on trained trait
    classifiers and output tabular YES/NO predictions with confidence values
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:180eb26e9d8644be1cd3015bb4ae37f148e7ae7f7998188cf8abbecc00e706fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# phenotype-prediction-table-compilation

## Summary

Apply a trained intracellular phenotype model (PICA SVM with libSVM backend) across all prokaryotic species in eggNOG 4.0 and compile the positive predictions into a species-level table. This reproduces the paper's screening that identified 169 obligate intracellular species (160 bacteria, 9 archaea).

## When to use

Use when a trained phenotypic trait model exists and you need to score every genome/species in a large database (e.g. eggNOG 4.0 core and periphery) and assemble the positives into a validated prediction table.

## When NOT to use

- The input is a raw genome FASTA rather than precomputed COG presence/absence trait profiles — train/sample extraction must happen first.
- You need predictions on incomplete metagenomic genome bins with no trait model trained on marker COGs; the paper cautions that reliable prediction requires roughly 60-70% genome completeness.
- The target trait has no pre-trained model; the paper notes performance for further ecologically important traits awaits sufficient training data.

## Inputs

- COG presence/absence trait profiles for all prokaryotic species in eggNOG 4.0 (core and periphery genomes)
- Feature subset of 39/40 universal marker COGs used by the phenotype model
- Pre-trained obligate intracellular phenotype model (PICA SVM / libSVM classifier)

## Outputs

- Species-level table of predicted intracellular organisms with positive/negative SVM labels
- Validated compilation of 169 predicted intracellular species split as 160 bacteria and 9 archaea (matching Figure 6 and Additional File 1 Table S3)

## How to apply

Load COG presence/absence trait profiles for all prokaryotic species from eggNOG 4.0 (core and periphery genomes) and restrict the feature space to the 39/40 universal marker COGs used by the phenotype model. Load the previously trained obligate intracellular model through the PICA support vector machine plug-in (libSVM backend) and score every species, assigning positive/negative intracellular labels. Compile all positive predictions into a species-level table split by bacterial and archaeal taxonomy. Retain known hard cases (e.g. the six Bartonella and Thermoprotei misclassifications) rather than filtering them out, and validate the table composition against the published reference counts.

## Related tools

- **PICA** (Framework that applies the trained phenotype model to score every prokaryotic species in the eggNOG 4.0 trait-profile matrix) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Plug-in that hosts the trained obligate intracellular SVM used to assign positive/negative labels to each species) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend SVM implementation behind the PICA plug-in that scores species against the intracellular trait model)
- **phenotrex** (Successor package to PICA that can run the same predict step on trained trait classifiers and output tabular YES/NO predictions with confidence values) — https://github.com/univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier intracellular.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- The compiled prediction table contains exactly 169 predicted intracellular species.
- The class split is exactly 160 bacteria and 9 archaea, matching Figure 6 and Additional File 1 Table S3.
- Known difficult taxa (six Bartonella and Thermoprotei misclassifications) are retained in the table, confirming they were not silently filtered.
- Every scored species was restricted to the 39/40 universal marker COGs feature space before SVM scoring — no extraneous COG features leaked in.
- Each compiled entry carries a positive/negative label from the trained SVM; per-species predictions are reproducible by re-running the model through the PICA/libSVM pipeline.

## Limitations

- Reliable trait prediction requires at least roughly 60-70% genome completeness, which constrains applicability to incomplete genome bins from metagenomes.
- Known misclassifications persist (six Bartonella and Thermoprotei genomes), so the table is not free of error even when validation counts match.
- PICA is superseded by the phenotrex package, which builds on scikit-learn/pandas and adds new classifiers; users reproducing the original eggNOG 4.0 screening should note the tool-generation difference.

## Evidence

- [abstract] New intracellular model: "We have established a new phenotypic model that predicts intracellular microorganisms."
- [intro] Obligate intracellular prediction scope: "We further present a novel model for prediction of obligate intracellular microorganisms."
- [abstract] SVM plug-in for large-scale screening: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [readme] Prediction output format (successor tool): [evidence span withheld — anchor status: external_doc]
- [discussion] Genome completeness limitation: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
