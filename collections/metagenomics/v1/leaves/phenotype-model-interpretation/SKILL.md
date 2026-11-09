---
name: phenotype-model-interpretation
description: Use when a trained linear-kernel phenotype model (e.g. PICA SVM on COG
  features) must be interpreted feature-wise, to test whether a trait is predicted
  by gene absence rather than gene presence.
when_to_use_negative:
- Input is a raw genome or metagenomic bin requiring a trait prediction — use the
  PICA/phenotrex prediction workflow instead of model interpretation
- No trained linear-kernel model exists for the trait of interest (e.g. insufficient
  training data), so there is no weight vector to rank
- Input is already a ranked feature table and only re-analysis or re-formatting is
  needed
tools:
- name: PICA
  role: Framework that produced the trained phenotype model whose weight vector is
    extracted and ranked
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Trains the libSVM linear-kernel SVM whose raw weight vector supplies the signed
    COG feature ranking
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: SVM implementation behind the linear-kernel model; its raw weight vector is
    the source of the feature ranking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:582fb2e74af9f905b026af54fedc53cc42ad277e19e54b6e081362b8f215cf17
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# phenotype-model-interpretation

## Summary

Extracts and ranks the signed SVM weight vector of a trained PICA/libSVM phenotype model (e.g. the obligate intracellular model trained on 97 labelled species) to identify which COG features — by presence or absence — drive the trait prediction, and validates the ranking against the published table (Additional File 1: Table S2).

## When to use

Use when a trained linear-kernel phenotype model (e.g. PICA SVM on COG features) must be interpreted feature-wise, to test whether a trait is predicted by gene absence rather than gene presence.

## When NOT to use

- Input is a raw genome or metagenomic bin requiring a trait prediction — use the PICA/phenotrex prediction workflow instead of model interpretation
- No trained linear-kernel model exists for the trait of interest (e.g. insufficient training data), so there is no weight vector to rank
- Input is already a ranked feature table and only re-analysis or re-formatting is needed

## Inputs

- Trained PICA intracellular phenotype model (libSVM linear kernel) trained on all 97 labelled species
- Published reference ranking of COG features (Additional File 1: Table S2)

## Outputs

- Signed ranked feature list of COGs ordered by absolute SVM weight (ranked_features.tsv), with weights and ranks

## How to apply

Load the trained intracellular phenotype model produced by the PICA SVM plug-in (libSVM, linear kernel) trained on all 97 labelled species. Extract the raw SVM weight vector and rank COG features by their absolute weights, retaining the sign so that negative weights encode predictors whose absence indicates the trait (critical for genome-reduction phenotypes). Compare the signed ranked COG list against the published ranking in Additional File 1: Table S2 to confirm agreement of the top entries, then tabulate features with weights and ranks and write ranked_features.tsv. The signed sign convention matters because for the intracellular trait all top 50 features are negative predictors, so interpretation must distinguish absence-of-gene evidence from presence-of-gene evidence.

## Related tools

- **PICA** (Framework that produced the trained phenotype model whose weight vector is extracted and ranked) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Trains the libSVM linear-kernel SVM whose raw weight vector supplies the signed COG feature ranking) — github.com/univieCUBE/PICA
- **libSVM** (SVM implementation behind the linear-kernel model; its raw weight vector is the source of the feature ranking)

## Evaluation signals

- Reproduced ranking shows all top 50 features are negative predictors (absence predictive for the obligate intracellular trait)
- Exactly 2 positive predictors appear among the top 100 features
- Exactly 14 positive predictors appear among the top 200 features
- Top entries of the reproduced ranked COG list agree with the published ranking in Additional File 1: Table S2
- ranked_features.tsv contains features, weights, and ranks and is internally consistent (ranks sorted by absolute weight, signs preserved)

## Limitations

- PICA is superseded by the phenotrex package; new work should target phenotrex models, and legacy PICA models may need conversion
- Sign interpretation is trait-dependent: for genome-reduction traits such as obligate intracellular lifestyle, strong predictors are gene absences, so naive top-weight (positive-only) reading of the model would be misleading
- Trait prediction (and thus model applicability) is reliable only for near-complete genomes — most traits can be reliably predicted in only 60-70% complete genomes

## Evidence

- [abstract] A new software tool facilitates the in-depth analysis of phenotype models: "A new software tool facilitates the in-depth analysis of phenotype models, which associate expected and unexpected protein functions with particular traits."
- [abstract] PICA framework role: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [readme] phenotrex stack: "Builds upon the battle-tested SciPy stack (`scikit-learn`, `pandas`, etc.)"
