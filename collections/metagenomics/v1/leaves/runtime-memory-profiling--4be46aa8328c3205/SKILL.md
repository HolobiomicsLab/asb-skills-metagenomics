---
name: runtime-memory-profiling
description: Use when you need to verify that a machine-learning phenotype-prediction
  pipeline remains tractable in runtime and memory as training-set size and feature
  dimensionality (e.g. genomes x COGs) grow toward large-scale databases.
when_to_use_negative:
- The goal is to assess predictive accuracy rather than computational tractability
  — random trait labels make accuracy meaningless here
- Input is real, already-annotated genome data for trait prediction rather than a
  scaling stress test
- The classifier being evaluated is not SVM-based via PICA (e.g. successor tooling
  such as phenotrex with XGBoost), since runtime behavior will not match the paper's
  benchmark
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: PICA
  role: Framework in which 5,000 virtual species are generated and cross-validation
    is run at progressively larger training-set sizes for runtime/memory measurement
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM classifier plug-in whose cross-validation runtime and peak memory are
    profiled during the scaling experiment
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Backend SVM library used by the PICA plug-in during the scaling runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:021955bc9168d9482228c94152b5461b4eb6cfb7689e6c5a1b92ad037f9b1b40
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# runtime-memory-profiling

## Summary

Benchmark the computational feasibility (wall-clock runtime and peak memory) of PICA's SVM-based phenotype prediction by scaling cross-validation runs up to 5,000 virtual species with ~200,000 COG features, as done to produce the tractability scaling curve in Figure 4 of the paper.

## When to use

Use when you need to verify that a machine-learning phenotype-prediction pipeline remains tractable in runtime and memory as training-set size and feature dimensionality (e.g. genomes x COGs) grow toward large-scale databases.

## When NOT to use

- The goal is to assess predictive accuracy rather than computational tractability — random trait labels make accuracy meaningless here
- Input is real, already-annotated genome data for trait prediction rather than a scaling stress test
- The classifier being evaluated is not SVM-based via PICA (e.g. successor tooling such as phenotrex with XGBoost), since runtime behavior will not match the paper's benchmark

## Inputs

- 5,000 virtual species generated in PICA, each with a genotype profile sized like an average eggNOG 4.0 genome, features sampled from ~200,000 virtual COGs
- Random trait labels assigned to each virtual species
- PICA configuration for cross-validation with the SVM plug-in (libSVM backend)
- A series of progressively larger training-set subsets up to the full 5,000-genome matrix

## Outputs

- Feasibility table of wall-clock runtime and peak memory consumption per training-set size
- Scaling curve of runtime/memory versus training-set size, corresponding to Figure 4 of the paper

## How to apply

Generate a simulated feature matrix with known scale parameters: 5,000 virtual species whose genotype profiles are sized like an average eggNOG 4.0 genome, sampling features from a pool of ~200,000 virtual COGs, with random trait labels so runtime, not accuracy, is measured. Run PICA cross-validation with the support vector machine plug-in (libSVM backend) on progressively larger subsets of the simulated genomes, scaling the training set upward toward the full 5,000-genome / ~200,000-feature matrix. At each training-set size, record wall-clock runtime and peak memory consumption of the cross-validation run. Tabulate runtime and memory as a function of set size into a feasibility table and plot the scaling curve to compare against the paper's Figure 4 trend.

## Related tools

- **PICA** (Framework in which 5,000 virtual species are generated and cross-validation is run at progressively larger training-set sizes for runtime/memory measurement) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM classifier plug-in whose cross-validation runtime and peak memory are profiled during the scaling experiment) — github.com/univieCUBE/PICA
- **libSVM** (Backend SVM library used by the PICA plug-in during the scaling runs)

## Evaluation signals

- The runtime/memory-versus-set-size table shows training on 5,000 genomes with ~200,000 COG features completes within the measured resource envelope
- The plotted scaling curve matches the tractability trend reported in Figure 4 of the paper
- Feature dimensionality of the simulated matrix is consistent with ~200,000 COGs per genome set
- Each cross-validation run uses the SVM plug-in with the libSVM backend, not another classifier
- Runtime and memory are recorded at every training-set size, not only at the largest scale point

## Limitations

- Trait labels are random, so the benchmark measures feasibility only — it says nothing about predictive accuracy at scale
- The paper notes most traits can be reliably predicted in only 60-70% complete genomes, a constraint orthogonal to but relevant when moving from simulated to real data
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)

## Evidence

- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [abstract] Our results suggest that the extended PICA framework can be used to automatically annotate phenotypes in near-complete microbial genome sequences: "Our results suggest that the extended PICA framework can be used to automatically annotate phenotypes in near-complete microbial genome sequences"
- [readme] Phenotrex is a component of the `PhenDB` web server, which performs phenotypic trait prediction on user-uploaded metagenomic bins.: [evidence span withheld — anchor status: not_found]
