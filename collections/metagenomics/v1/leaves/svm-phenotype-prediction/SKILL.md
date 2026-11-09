---
name: svm-phenotype-prediction
description: Use when predicting a binary microbial phenotypic trait (e.g. obligate
  intracellular lifestyle) from comparative-genomics feature matrices such as COG
  presence/absence vectors, or when screening whole genome databases like eggNOG for
  species carrying a trained trait.
when_to_use_negative:
- Genomes fail the marker-COG completeness screen (fewer than 39/40 universal marker
  COGs present) — predictions are unreliable below ~60-70% completeness
- The target trait is not characterized by genome reduction / gene absence patterns
  that presence-absence COG vectors can capture
- Sufficient training data with labeled genomes for the trait are not available (the
  article notes pending performance for further ecologically important traits awaiting
  training data)
edam_operation: http://edamontology.org/operation_3094
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: PICA
  role: Framework for building and applying comparative-genomics phenotype prediction
    models on COG presence/absence data
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM plug-in used to train the obligate intracellular phenotype classifier
    and score eggNOG 4.0 species
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend SVM library invoked by the PICA support vector machine plug-in for
    training and prediction
- name: phenotrex
  role: Successor package to PICA providing end-to-end microbial phenotypic trait
    prediction with a CLI predict command and pre-trained models via PhenDB
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:b2cd7a1f22fb66ff7d9e7b4e07744724343d45000cc6e4c1de0343bfa64709fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_3094
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# svm-phenotype-prediction

## Summary

Train and apply support vector machine phenotype models on COG presence/absence profiles using the PICA framework (libSVM backend), e.g. the obligate intracellular lifestyle model that reached 0.997 ± 0.010 cross-validated balanced accuracy on 97 complete bacterial genomes.

## When to use

Use when predicting a binary microbial phenotypic trait (e.g. obligate intracellular lifestyle) from comparative-genomics feature matrices such as COG presence/absence vectors, or when screening whole genome databases like eggNOG for species carrying a trained trait.

## When NOT to use

- Genomes fail the marker-COG completeness screen (fewer than 39/40 universal marker COGs present) — predictions are unreliable below ~60-70% completeness
- The target trait is not characterized by genome reduction / gene absence patterns that presence-absence COG vectors can capture
- Sufficient training data with labeled genomes for the trait are not available (the article notes pending performance for further ecologically important traits awaiting training data)

## Inputs

- COG presence/absence feature matrix for complete bacterial genomes (e.g. 97 genomes × 30,455 COGs) with trait labels from Additional File 1 Table S1
- Genomes passing the marker-COG completeness filter (39/40 universal marker COGs)
- COG presence/absence trait profiles for all prokaryotic species in eggNOG 4.0 (core and periphery genomes)
- A previously trained PICA/libSVM phenotype model (serialized classifier)

## Outputs

- Trained SVM phenotype model for the trait (e.g. obligate intracellular lifestyle)
- Per-fold cross-validation predictions with balanced accuracy per fold (cv_results.csv)
- Aggregated performance metric: mean balanced accuracy ± SD (0.997 ± 0.010)
- Species-level table of positive predictions from database screening (169 species: 160 bacteria, 9 archaea)
- Tabular predictions with YES/NO trait presence and confidence values (0.5–1.0) when using the phenotrex successor

## How to apply

Build a COG presence/absence feature matrix (the intracellular model used 30,455 COGs over 97 complete genomes: 43 obligate intracellular, 6 facultative, 48 free-living) and attach trait labels. Apply the marker-COG completeness filter (39/40 universal marker COGs) so only sufficiently complete genomes enter training, since most traits are reliably predictable only in 60–70% complete genomes. Train via the PICA support vector machine plug-in backed by libSVM, then run 5-fold cross-validation with 10 replicates, aggregating balanced accuracies into mean ± SD (target 0.997 ± 0.010). For database-scale screening, restrict eggNOG 4.0 core and periphery genome profiles to the 39/40 marker COGs, load the trained model, and score every species to obtain positive/negative labels (expect 169 predicted intracellular species: 160 bacteria, 9 archaea). Validation requires reproducing the reported accuracy, the misclassification pattern (Cand. Pelagibacter sp. IMCC9063 misclassified in 3/10 folds), and the class composition of positive predictions.

## Related tools

- **PICA** (Framework for building and applying comparative-genomics phenotype prediction models on COG presence/absence data) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM plug-in used to train the obligate intracellular phenotype classifier and score eggNOG 4.0 species) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend SVM library invoked by the PICA support vector machine plug-in for training and prediction)
- **phenotrex** (Successor package to PICA providing end-to-end microbial phenotypic trait prediction with a CLI predict command and pre-trained models via PhenDB) — https://github.com/univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Mean cross-validated balanced accuracy across 10 replicates of 5-fold CV matches the reported 0.997 ± 0.010
- All positives and negatives are predicted correctly in all folds except Cand. Pelagibacter sp. IMCC9063, which is misclassified as obligate intracellular in exactly 3 of 10 folds
- Applying the trained model to eggNOG 4.0 core and periphery genomes yields exactly 169 predicted intracellular species split as 160 bacteria and 9 archaea (retaining the six Bartonella and Thermoprotei misclassifications)
- Every genome entering training passes the 39/40 universal marker COG completeness filter
- Per-fold predictions and the summary metric are written to cv_results.csv with one predicted label per genome per fold

## Limitations

- Most traits can be reliably predicted only in 60-70% complete genomes, so incomplete genome bins may give unreliable predictions
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)
- Trait prediction relies on gene absence in reduced genomes, so lineages with convergently reduced genomes (e.g. free-living Cand. Pelagibacter sp. IMCC9063) can be misclassified as obligate intracellular
- PICA is superseded by the phenotrex package, which the README recommends for new work

## Evidence

- [abstract] PICA SVM accuracy: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [abstract] SVM plugin extension: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [discussion] genome completeness limitation: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] phenotrex successor: "Please checkout the new `phenotrex` package, which succeeds PICA"
- [readme] phenotrex predict invocation: [evidence span withheld — anchor status: external_doc]
