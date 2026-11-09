---
name: large-scale-cog-feature-matrix-handling
description: Use when you must demonstrate that an SVM-based phenotype-prediction
  pipeline remains computationally feasible (runtime and memory) at very large training-set
  sizes, e.g. thousands of genomes with hundreds of thousands of COG features.
when_to_use_negative:
- When the goal is accurate phenotype prediction rather than computational scaling
  — the virtual species carry random trait labels, so this skill yields no predictive-performance
  insight
- When the training set is small (far below 5,000 genomes / 200,000 features), the
  scaling benchmark adds no information over simply running the prediction
- When the input is already a real annotated feature table of actual genomes — this
  skill's first step generates simulated data, not curation of real COG annotations
tools:
- name: PICA
  role: Framework used to generate the 5,000 virtual species and run cross-validation
    over scaling training subsets
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM plug-in whose computational tractability (runtime, memory) is being benchmarked
    at large scale
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend solver used by the PICA SVM plug-in during the scaling cross-validation
    runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:4d7f98f605428a45184150aa18c41feeaf38d109586f97a22349b7b07fe6d997
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# large-scale-cog-feature-matrix-handling

## Summary

A computational feasibility benchmark skill that stress-tests SVM-based phenotypic trait prediction (PICA with libSVM backend) on simulated genotype-by-COG feature matrices scaling up to 5,000 genomes × ~200,000 COGs, recording wall-clock runtime and peak memory to demonstrate large-scale tractability.

## When to use

Use when you must demonstrate that an SVM-based phenotype-prediction pipeline remains computationally feasible (runtime and memory) at very large training-set sizes, e.g. thousands of genomes with hundreds of thousands of COG features.

## When NOT to use

- When the goal is accurate phenotype prediction rather than computational scaling — the virtual species carry random trait labels, so this skill yields no predictive-performance insight
- When the training set is small (far below 5,000 genomes / 200,000 features), the scaling benchmark adds no information over simply running the prediction
- When the input is already a real annotated feature table of actual genomes — this skill's first step generates simulated data, not curation of real COG annotations

## Inputs

- Pool of ~200,000 virtual COG feature identifiers
- 5,000 virtual species genotype profiles sized like an average eggNOG 4.0 genome (random features drawn from the COG pool)
- Random binary trait labels assigned to each virtual species
- Progressively larger training subsets of the simulated genomes

## Outputs

- Feasibility table of wall-clock runtime and peak memory as a function of training-set size
- Scaling curve (runtime/memory versus set size) reproducing Figure 4 of the paper
- Demonstrated resource envelope for 5,000-genome × ~200,000-COG cross-validation

## How to apply

Generate 5,000 virtual species in PICA, each with a genotype profile sized like an average eggNOG 4.0 genome, sampling features at random from a pool of ~200,000 virtual COGs and assigning random trait labels. Run PICA cross-validation with the SVM plug-in (libSVM backend) on progressively larger subsets of the simulated genomes, scaling the training matrix upward toward the full 5,000-genome × ~200,000-feature size. For each training-set size, record the wall-clock runtime and peak memory consumption of the cross-validation run. Tabulate runtime and memory versus set size into a feasibility table and plot the scaling curve corresponding to Figure 4 of the paper. The rationale is that randomized virtual species isolate computational scaling from biological signal, since trait labels are random, so any infeasibility reflects the SVM pipeline itself rather than trait predictability.

## Related tools

- **PICA** (Framework used to generate the 5,000 virtual species and run cross-validation over scaling training subsets) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM plug-in whose computational tractability (runtime, memory) is being benchmarked at large scale) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend solver used by the PICA SVM plug-in during the scaling cross-validation runs)

## Examples

```
pip install phenotrex[fasta]
```

## Evaluation signals

- Cross-validation completes at the full 5,000-genome × ~200,000-COG training-set size without exhausting runtime or memory
- The runtime-versus-set-size and memory-versus-set-size tables are complete for every measured subset size with no missing entries
- The plotted scaling curve matches the tractability trend reported in Figure 4 of the paper
- Virtual species features are confirmed to be drawn from a pool of ~200,000 COGs at eggNOG 4.0-average genome size, and trait labels are random
- Rerunning the benchmark reproduces runtime/memory measurements within the same resource envelope

## Limitations

- The benchmark uses random trait labels on virtual species, so it validates computational tractability only, not predictive accuracy
- Simulated genome profiles approximate (rather than replicate) real eggNOG 4.0 feature distributions
- PICA itself is superseded by the phenotrex package, so the benchmark would need adaptation to reproduce with the successor stack

## Evidence

- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases"
- [intro] It includes plug-ins for CPAR and libSVM [19] among others.: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] Builds upon the battle-tested SciPy stack (`scikit-learn`, `pandas`, etc.): "Builds upon the battle-tested SciPy stack (`scikit-learn`, `pandas`, etc.)"
