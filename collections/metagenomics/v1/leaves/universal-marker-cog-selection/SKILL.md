---
name: universal-marker-cog-selection
description: Use when scoring every prokaryotic species in a COG-annotated genome
  database with a trained phenotype model whose feature space is limited to universal
  marker COGs, or when building prediction inputs for incomplete genomes where a full
  COG profile would be inconsistent.
when_to_use_negative:
- The target model was trained on the full COG presence/absence profile rather than
  a marker-COG subset — restricting features would discard signal the model expects.
- The goal is de novo identification of trait-associated functions rather than applying
  a pre-trained model; then the full feature space is needed for model analysis/explanation.
- Genomes lack COG annotation entirely, so no marker-COG vector can be constructed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PICA
  role: Framework that consumes the marker-COG-restricted trait profiles and applies
    the trained phenotype model via its SVM plug-in
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM classifier plug-in used to score species on the marker-COG feature space
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend used by the PICA SVM plug-in for classification
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:0d0a7759d5d262e1ea697815359ef5ce1c74325369e0ec4e58302d807ca1ae2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# universal-marker-cog-selection

## Summary

Restrict the COG presence/absence trait profiles from a large genome database (e.g. eggNOG 4.0 core and periphery genomes) to the small set of ~39/40 universal marker COGs used by the PICA phenotype model, so that all species — including incomplete ones — can be scored on a comparable feature space.

## When to use

Use when scoring every prokaryotic species in a COG-annotated genome database with a trained phenotype model whose feature space is limited to universal marker COGs, or when building prediction inputs for incomplete genomes where a full COG profile would be inconsistent.

## When NOT to use

- The target model was trained on the full COG presence/absence profile rather than a marker-COG subset — restricting features would discard signal the model expects.
- The goal is de novo identification of trait-associated functions rather than applying a pre-trained model; then the full feature space is needed for model analysis/explanation.
- Genomes lack COG annotation entirely, so no marker-COG vector can be constructed.

## Inputs

- COG presence/absence trait profiles for all prokaryotic species from a genome database (e.g. eggNOG 4.0 core and periphery genomes)
- Trained PICA phenotype model (e.g. obligate intracellular SVM) with its known universal marker COG feature set (39/40 COGs)

## Outputs

- Reduced species x marker-COG feature matrix containing only the 39/40 universal marker COG columns
- Model-ready feature vectors per species for downstream SVM scoring (positive/negative intracellular labels)

## How to apply

Load the COG presence/absence trait profiles for all prokaryotic species from the genome database (e.g. eggNOG 4.0, covering core and periphery genomes). Restrict the feature matrix to the 39/40 universal marker COGs that define the trained intracellular phenotype model, discarding non-marker COG columns so that every genome is represented by the same marker-derived feature vector. Rationale: the article shows that phenotypic trait prediction relies on generic COG-based comparative features and remains stable even for only 60–70% complete genomes, so a compact universal-marker feature space lets an SVM (PICA plug-in with libSVM backend) be applied uniformly across the full database, including draft and periphery genomes. The filtered vectors are then scored by the trained model (e.g. obligate intracellular SVM) to assign positive/negative labels per species. Decision point: verify that the retained marker set matches the model's training features exactly; applying a model to features outside its training space invalidates predictions.

## Related tools

- **PICA** (Framework that consumes the marker-COG-restricted trait profiles and applies the trained phenotype model via its SVM plug-in) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM classifier plug-in used to score species on the marker-COG feature space) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend used by the PICA SVM plug-in for classification)

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- The reduced feature matrix contains exactly the 39/40 universal marker COG columns used by the trained model — no extra or missing markers.
- Every prokaryotic species in the database (core and periphery) yields a complete marker-COG vector with no species dropped during restriction.
- Downstream SVM application to the restricted vectors reproduces the expected prediction composition, e.g. exactly 169 predicted intracellular species split 160 bacteria / 9 archaea for the eggNOG 4.0 screen.
- Genomes at ~60–70% completeness still produce reliable predictions, consistent with the paper's completeness finding — incomplete genomes should not be excluded at the marker-filter stage.

## Limitations

- Reliable trait prediction requires at least 60–70% genome completeness; below that, marker-COG profiles become too sparse for reasonable predictions.
- Predictions retain known misclassifications (e.g. the six Bartonella and Thermoprotei misclassifications were retained in the 169-species intracellular screen).
- Performance for further ecologically important traits is pending availability of sufficient training data.

## Evidence

- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [discussion] Most traits can be reliably predicted in only 60-70% complete genomes: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [intro] Generic computational methods based on large databases of COGs: "Generic computational methods, namely those based on large databases of COGs, will most likely represent a first layer of methods for trait prediction."
- [intro] It includes plug-ins for CPAR and libSVM [19] among others.: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv: [evidence span withheld — anchor status: external_doc]
