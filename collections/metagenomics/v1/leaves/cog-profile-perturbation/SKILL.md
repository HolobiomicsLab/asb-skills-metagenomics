---
name: cog-profile-perturbation
description: Use when assessing the robustness of COG/SVM-based phenotypic trait predictors
  to genome incompleteness, e.g. before applying trait models to incomplete metagenomic
  bins.
when_to_use_negative:
- The input genomes are already complete reference genomes and the goal is final trait
  annotation rather than robustness assessment — near-complete genomes can be annotated
  directly with PICA without perturbation
- The features are not COG presence/absence profiles (e.g. raw FASTQ reads or un-annotated
  contigs); first map genes to eggNOG/COG to build profiles before thinning
- The prediction pipeline is not profile-based (e.g. sequence-homology or k-mer methods);
  removing random COG entries would not represent its natural incompleteness mode
tools:
- name: PICA
  role: Framework providing the flat-file COG profile format used for the training
    profiles and the thinned incomplete test profiles
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Consumes the thinned incomplete profiles as test sets to evaluate SVM trait-prediction
    performance at each completeness level
  repo: github.com/univieCUBE/PICA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:e93d7082887c26f7cc4fb537c118f18c6ee9c483b6b86759664425a144c88ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cog-profile-perturbation

## Summary

Simulates incomplete genomes by randomly deleting x% (x = 10–90, in steps of 10) of the eggNOG 4.0 COG presence/absence entries per genome profile (3 replicates per level), producing PICA-compatible thinned profiles used to test SVM-based phenotypic trait prediction on partial genomes such as metagenomic bins.

## When to use

Use when assessing the robustness of COG/SVM-based phenotypic trait predictors to genome incompleteness, e.g. before applying trait models to incomplete metagenomic bins.

## When NOT to use

- The input genomes are already complete reference genomes and the goal is final trait annotation rather than robustness assessment — near-complete genomes can be annotated directly with PICA without perturbation
- The features are not COG presence/absence profiles (e.g. raw FASTQ reads or un-annotated contigs); first map genes to eggNOG/COG to build profiles before thinning
- The prediction pipeline is not profile-based (e.g. sequence-homology or k-mer methods); removing random COG entries would not represent its natural incompleteness mode

## Inputs

- eggNOG 4.0 COG presence/absence profiles per genome (PICA flat-file profile format) for genomes with known phenotypic traits
- Trained PICA SVM phenotypic trait models
- Fixed random seed for reproducible sampling

## Outputs

- 27 incomplete COG profile files (9 completeness levels x 3 replicates), named e.g. incomplete_90_1 ... incomplete_10_3 in PICA-compatible format
- Per-level SVM prediction performance of trait models on thinned test genomes, revealing the completeness threshold (reported as 60-70%) below which trait prediction degrades

## How to apply

Load eggNOG 4.0 COG presence/absence profiles for genomes with known traits in PICA's flat-file profile format. For each removal level x in {10, 20, 30, 40, 50, 60, 70, 80, 90}, randomly delete x% of the total COG entries in each genome's profile using a fixed random seed, and repeat 3 times per level to obtain independent replicates. Write each thinned profile to a file named by completeness level and replicate (e.g. incomplete_90_1, incomplete_90_2, incomplete_90_3), keeping the PICA-compatible format so the PICA SVM plug-in can consume them directly as test sets. The rationale is that metagenomic bins are typically partial; the paper reports most traits remain reliably predictable in only 60–70% complete genomes, which bounds reasonable bin-level application. Validate that exactly 27 files (9 levels × 3 replicates) exist, each containing exactly (100 − x)% of the original per-genome COG count with no novel COGs.

## Related tools

- **PICA** (Framework providing the flat-file COG profile format used for the training profiles and the thinned incomplete test profiles) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Consumes the thinned incomplete profiles as test sets to evaluate SVM trait-prediction performance at each completeness level) — github.com/univieCUBE/PICA

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl incomplete_90_1.fna > predictions.tsv
```

## Evaluation signals

- Exactly 27 incomplete-profile files exist (9 removal levels x 3 replicates), named by completeness level and replicate (e.g. incomplete_90_1)
- Each file contains exactly (100 - x)% of the original COG count per genome
- No COGs appear in a thinned profile that were absent from the original genome profile
- Re-running with the same fixed random seed reproduces identical thinned profiles (replicate-level reproducibility)
- Thinned profiles still load in PICA's flat-file profile format and are directly accepted by the PICA SVM plug-in as test sets

## Limitations

- Most traits are only reliably predictable in 60-70% complete genomes, so predictions in bins below this completeness are not trustworthy
- Random COG removal may not mimic real binning incompleteness, which is typically non-random (e.g. loss of gene clusters or contigs)
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)

## Evidence

- [methods] Incomplete test genomes were simulated by random removal of x percent of the total number of COGs per genome, for x in [10, 20, ..., 90] with 3 replicates: [evidence span withheld — anchor status: not_found]
- [abstract] Most of the traits can be reliably predicted in only 60-70% complete genomes.: "Most of the traits can be reliably predicted in only 60-70% complete genomes."
- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [readme] $ phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv: [evidence span withheld — anchor status: external_doc]
