---
name: virtual-species-genome-simulation
description: Use when you need to benchmark the computational tractability (wall-clock
  runtime and peak memory) of a trait-prediction pipeline at training-set sizes beyond
  your real labeled data, by simulating genomes with realistic feature-matrix dimensions.
when_to_use_negative:
- Your goal is to measure predictive accuracy on real genomes with real trait labels
  — this simulation uses random labels and yields no biological performance estimate
- Your feature matrix is far smaller than the eggNOG 4.0-scale dimensions (~200,000
  COGs); smaller feature spaces will not exercise the scalability limits this benchmark
  targets
- You are using the successor phenotrex/scikit-learn stack rather than PICA's libSVM
  plug-in; the runtime envelope measured here applies to PICA's SVM backend, not the
  newer classifiers (e.g. XGBoost)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PICA
  role: Framework in which the 5,000 virtual species are generated and cross-validation
    scaling runs are executed
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM classifier plug-in whose runtime and memory are benchmarked across training-set
    sizes
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Backend SVM library invoked by the PICA plug-in during the cross-validation
    scaling runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:5fa6cd098dfde700683478c27ea05115f9b5150a37a99e59887406d1b38f35ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# virtual-species-genome-simulation

## Summary

Generates synthetic 'virtual species' genotype profiles inside PICA by randomly sampling features from a pool of ~200,000 virtual COGs and assigning random trait labels, then benchmarks SVM cross-validation runtime and memory as training sets scale toward 5,000 genomes. This establishes the computational feasibility envelope of PICA's libSVM-backed phenotype prediction at large-genome-database scale (Figure 4 of the paper).

## When to use

Use when you need to benchmark the computational tractability (wall-clock runtime and peak memory) of a trait-prediction pipeline at training-set sizes beyond your real labeled data, by simulating genomes with realistic feature-matrix dimensions.

## When NOT to use

- Your goal is to measure predictive accuracy on real genomes with real trait labels — this simulation uses random labels and yields no biological performance estimate
- Your feature matrix is far smaller than the eggNOG 4.0-scale dimensions (~200,000 COGs); smaller feature spaces will not exercise the scalability limits this benchmark targets
- You are using the successor phenotrex/scikit-learn stack rather than PICA's libSVM plug-in; the runtime envelope measured here applies to PICA's SVM backend, not the newer classifiers (e.g. XGBoost)

## Inputs

- Pool of ~200,000 virtual COGs (synthetic feature identifiers)
- Target number of virtual species (5,000) with genotype profiles sized like an average eggNOG 4.0 genome
- Random trait labels assigned per virtual species
- Progressively larger training-set sizes for the scaling sweep

## Outputs

- Simulated 5,000-genome x ~200,000-COG genotype/trait matrix in PICA's internal format
- Per-size cross-validation wall-clock runtime measurements
- Per-size peak memory consumption measurements
- Feasibility table and scaling plot (runtime/memory vs. training-set size, corresponding to Figure 4)

## How to apply

First, size each virtual species genotype to match an average eggNOG 4.0 genome, sampling its features at random from a pool of ~200,000 virtual COGs, and assign random trait labels so that no real biological signal confounds the timing measurements. Second, run PICA cross-validation with the SVM plug-in (libSVM backend) on progressively larger subsets of the 5,000 simulated genomes, scaling the training set upward toward the full 5,000-genome x ~200,000-feature matrix. Third, at each training-set size record the wall-clock runtime and peak memory consumption of the cross-validation run. Finally, tabulate runtime and memory versus set size into a feasibility table and plot the scaling curve corresponding to Figure 4. The random-label design is deliberate: it isolates computational cost from predictive performance, which is reported separately in the paper.

## Related tools

- **PICA** (Framework in which the 5,000 virtual species are generated and cross-validation scaling runs are executed) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM classifier plug-in whose runtime and memory are benchmarked across training-set sizes) — github.com/univieCUBE/PICA
- **libSVM** (Backend SVM library invoked by the PICA plug-in during the cross-validation scaling runs)

## Evaluation signals

- The simulated genotype matrix reaches the target dimensions: 5,000 virtual species with features drawn from ~200,000 virtual COGs, sized like an average eggNOG 4.0 genome
- The runtime/memory-versus-set-size table shows the full 5,000-genome / ~200,000-COG cross-validation completing within the measured resource envelope
- The plotted scaling curve reproduces the tractability trend of Figure 4 (PICA cross-validation remains computationally feasible at 5,000 genomes)
- Trait labels are random, so cross-validation accuracy should hover at chance level — any strong accuracy signals a simulation bug
- Each scaling point corresponds to a monotonically increasing training-set size with paired runtime and peak-memory readings

## Limitations

- Random trait labels mean results capture computational cost only, not predictive power; biological performance must be assessed on real labeled genomes
- Features are drawn at random from ~200,000 virtual COGs, so the simulation does not reflect real COG frequency distributions or genome-reduction signatures
- The measured envelope is specific to the libSVM-backed PICA plug-in; the paper's successor phenotrex uses the SciPy stack (scikit-learn, pandas) and new classifiers (e.g. XGBoost), with different performance characteristics
- Reliable trait prediction on real data requires at least 60-70% genome completeness, a constraint the virtual-species simulation does not model

## Evidence

- [abstract] PICA framework facilitates SVM scalability benchmarking: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [abstract] SVM plug-in extended for large-scale genome databases: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [intro] libSVM is the backend invoked by the PICA plug-in: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] Successor stack uses the SciPy stack with new classifiers: "Builds upon the battle-tested SciPy stack (`scikit-learn`, `pandas`, etc.)"
