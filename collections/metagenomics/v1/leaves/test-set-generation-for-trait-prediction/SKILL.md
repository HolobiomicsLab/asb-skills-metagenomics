---
name: test-set-generation-for-trait-prediction
description: Use when assessing SVM phenotypic trait prediction robustness on incomplete
  genomes or metagenomic bins by generating randomly thinned COG profiles at controlled
  completeness levels.
when_to_use_negative:
- Input genomes are already near-complete annotated genome sequences for which the
  PICA framework can annotate phenotypes directly without simulated thinning
- The analysis target is not SVM-based trait prediction from COG profiles but e.g.
  a different feature type (e.g. k-mer or gene-content models outside the PICA profile
  format)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework providing the flat-file COG profile format in which the incomplete
    test profiles are written and consumed
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Consumes the simulated incomplete profiles as test sets for SVM-based phenotypic
    trait prediction
  repo: https://github.com/univieCUBE/PICA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:2ea1793bbd81daed1fdfbf48d72663c7dba5208e385ac314a913704ed553d439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# test-set-generation-for-trait-prediction

## Summary

Simulates genome incompleteness by randomly removing 10–90% of COG entries from eggNOG 4.0 COG presence/absence profiles (3 replicates per level), producing PICA-compatible incomplete test sets to evaluate how well SVM-based phenotypic trait prediction tolerates metagenomic bins of reduced completeness.

## When to use

Use when assessing SVM phenotypic trait prediction robustness on incomplete genomes or metagenomic bins by generating randomly thinned COG profiles at controlled completeness levels.

## When NOT to use

- Input genomes are already near-complete annotated genome sequences for which the PICA framework can annotate phenotypes directly without simulated thinning
- The analysis target is not SVM-based trait prediction from COG profiles but e.g. a different feature type (e.g. k-mer or gene-content models outside the PICA profile format)

## Inputs

- eggNOG 4.0 COG presence/absence profiles in PICA flat-file profile format
- Genome-to-trait assignments for the example traits
- Removal levels (x = 10–90 in steps of 10) and number of replicates (3)

## Outputs

- 27 incomplete profile files (9 completeness levels x 3 replicates), e.g. incomplete_90_1, incomplete_90_2, incomplete_90_3
- PICA-compatible test-set profiles consumable by the PICA SVM plug-in

## How to apply

Load eggNOG 4.0 COG presence/absence profiles in the PICA flat-file profile format for genomes with known example traits. For each removal level x in {10, 20, ..., 90}, randomly sample and delete x percent of the total COG entries per genome using a fixed random seed, and repeat 3 times per level for independent replicates. Write each thinned profile to a PICA-compatible file named by level and replicate (e.g. incomplete_90_1 through incomplete_90_3) so they can be fed directly to the PICA SVM plug-in as test sets. The rationale is that metagenomic bins are typically incomplete, so evaluation must reflect partial genomes rather than near-complete references.

## Related tools

- **PICA** (Framework providing the flat-file COG profile format in which the incomplete test profiles are written and consumed) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Consumes the simulated incomplete profiles as test sets for SVM-based phenotypic trait prediction) — https://github.com/univieCUBE/PICA

## Evaluation signals

- Exactly 27 incomplete-profile files exist (9 removal levels x 3 replicates)
- Each incomplete profile at level x contains exactly (100 - x) percent of the original per-genome COG count
- No COGs appear in an incomplete profile that were absent from the original genome profile
- Incomplete profiles remain parseable by the PICA SVM plug-in as test sets (PICA-compatible format preserved)
- Fixed random seeds allow exact regeneration of a given replicate across runs

## Limitations

- Most traits can only be reliably predicted in 60-70% complete genomes, so predictions below that completeness are not expected to be reliable
- Random COG removal is a simulation of incompleteness and may not capture real metagenomic bin biases (non-random gene loss)
- Performance for further ecologically important traits is pending availability of sufficient training data

## Evidence

- [methods] Incomplete test genomes were simulated by random removal of x percent of the total number of COGs per genome: [evidence span withheld — anchor status: not_found]
- [discussion] Most of the traits can be reliably predicted in only 60-70% complete genomes: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
