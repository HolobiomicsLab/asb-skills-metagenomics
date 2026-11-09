---
name: cog-profile-feature-filtering
description: Use when applying a trained phenotype model (e.g. obligate intracellular
  prediction) to new species whose genomes are summarized as COG presence/absence
  profiles, and the model expects only its specific universal marker COG features
  (e.g. the 39/40 marker COGs) rather than the full eggNOG 4.
when_to_use_negative:
- Input is already a filtered, model-compatible feature table containing only the
  universal marker COGs — re-filtering may drop or misalign required features.
- Genome completeness is below roughly 60-70%, since the paper reports most traits
  can only be reliably predicted in 60-70% complete genomes, so COG profiles will
  be too incomplete for reliable trait prediction.
- The goal is trait prediction from raw sequences or metagenomic bins rather than
  from COG presence/absence profiles of known species.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework that hosts the trained phenotype models and applies them to COG
    trait profiles for phenotypic trait prediction
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM plug-in used to load and score the trained intracellular phenotype model
    against the filtered COG feature vectors
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend library underlying the PICA SVM plug-in for classifier execution
- name: phenotrex
  role: Successor package to PICA for end-to-end microbial phenotypic trait prediction,
    useful for newer re-implementations of the same workflow
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:dce9d352e6edb44833b404e3c9538b7161a8c69a8d0bff0612b69f082205b9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# COG profile feature filtering for phenotype prediction

## Summary

Restricts COG presence/absence trait profiles (e.g. from eggNOG 4.0 core and periphery genomes) to the universal marker COGs used by a trained phenotype model, so that the PICA/libSVM classifier can score species consistently. This is the feature-space alignment step that enables applying a trained intracellular phenotype model across large genome databases.

## When to use

Use when applying a trained phenotype model (e.g. obligate intracellular prediction) to new species whose genomes are summarized as COG presence/absence profiles, and the model expects only its specific universal marker COG features (e.g. the 39/40 marker COGs) rather than the full eggNOG 4.0 COG space.

## When NOT to use

- Input is already a filtered, model-compatible feature table containing only the universal marker COGs — re-filtering may drop or misalign required features.
- Genome completeness is below roughly 60-70%, since the paper reports most traits can only be reliably predicted in 60-70% complete genomes, so COG profiles will be too incomplete for reliable trait prediction.
- The goal is trait prediction from raw sequences or metagenomic bins rather than from COG presence/absence profiles of known species.

## Inputs

- COG presence/absence trait profiles for all prokaryotic species from the eggNOG 4.0 database (core and periphery genomes)
- List of the 39/40 universal marker COGs used by the trained phenotype model
- Trained intracellular phenotype model (PICA SVM model file)

## Outputs

- Filtered COG feature matrix (species x 39/40 marker COGs) matching the model's feature space
- Species-level table of positive intracellular predictions with SVM positive/negative labels
- Taxonomic composition summary (bacteria vs archaea counts)

## How to apply

Load the COG presence/absence trait profiles for all prokaryotic species from the eggNOG 4.0 database, covering both core and periphery genomes. Restrict the feature space to exactly the 39/40 universal marker COGs that the phenotype model was trained on, dropping all other COG columns so the input vectors match the classifier's expected dimensionality and ordering. Load the trained intracellular phenotype model through the PICA support vector machine plug-in with the libSVM backend, then score every species and assign positive/negative intracellular labels. Compile positive predictions into a species-level table and verify the taxonomic composition against the published reference outcome (169 species: 160 bacteria, 9 archaea).

## Related tools

- **PICA** (Framework that hosts the trained phenotype models and applies them to COG trait profiles for phenotypic trait prediction) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM plug-in used to load and score the trained intracellular phenotype model against the filtered COG feature vectors) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend library underlying the PICA SVM plug-in for classifier execution)
- **phenotrex** (Successor package to PICA for end-to-end microbial phenotypic trait prediction, useful for newer re-implementations of the same workflow) — https://github.com/univieCUBE/phenotrex

## Examples

```
$ phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- The filtered feature matrix contains exactly the 39/40 universal marker COGs, with feature count and column ordering matching the trained model's expectations.
- The compiled prediction table contains exactly 169 predicted intracellular species.
- The class split of the positive predictions is 160 bacteria and 9 archaea, matching Figure 6 and Additional File 1 Table S3.
- The known six Bartonella and Thermoprotei misclassifications are retained in the prediction output, confirming the model and feature filtering were applied identically to the reference run.
- Every species in the eggNOG 4.0 prokaryotic set receives a positive or negative intracellular label (no missing or dropped predictions).

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness; the paper notes most traits can be reliably predicted in only 60-70% complete genomes, which bounds applicability to partial genomes and metagenome bins.
- Known systematic misclassifications exist: the reference outcome retains six Bartonella and Thermoprotei misclassifications, so exact reproduction requires preserving them.
- Performance for further ecologically important traits was pending availability of sufficient training data at the time of publication.
- The PICA workflow is succeeded by the phenotrex package (pip installable, Python 3.7+), so PICA-based reproduction may require the original environment.

## Evidence

- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [intro] SVM plug-in and libSVM backend: "It includes plug-ins for CPAR and libSVM [19] among others."
- [discussion] Genome completeness requirement: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] Phenotrex CLI usage for applying a trained trait model to a genome: [evidence span withheld — anchor status: external_doc]
