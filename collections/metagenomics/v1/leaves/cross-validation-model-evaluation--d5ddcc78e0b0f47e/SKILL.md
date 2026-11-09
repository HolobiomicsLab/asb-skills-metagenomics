---
name: cross-validation-model-evaluation
description: 'Use when reproducing or verifying the Character Predictor of MicroPIE:
  trained SVM models over labelled taxonomic-description sentences must be evaluated
  with k-fold cross validation and compared to reported precision/recall.'
when_to_use_negative:
- The task is end-to-end phenotypic matrix extraction from new taxonomic descriptions
  rather than evaluating classifier accuracy (run MicroPIE normally instead)
- The text domain is not prokaryotic taxonomic descriptions; the 42-character SVM
  models and term lists are tailored to physiological/ecological trait sentences,
  not morphology-focused descriptions
- No labelled sentence instances are available, since supervised cross validation
  requires ground-truth labels for each sentence
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: MicroPIE 0.1.0
  role: Source of the labelled sentence corpus, SVM character models, and Term Lists
    used for the cross-validation evaluation
  repo: github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Supervised classification algorithm behind the 42 per-character LibSVM sentence
    classifiers being cross-validated
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:f6ff012d58f06e8a2c289716bc198e394cbbac5c83a860ed138b5ca32ed9029a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# cross-validation-model-evaluation

## Summary

Evaluate supervised sentence classifiers (per-character LibSVM models in MicroPIE 0.1.0) by 10-fold cross validation, recording precision and recall per character and averaging across all 42 microbial phenotypic characters.

## When to use

Use when reproducing or verifying the Character Predictor of MicroPIE: trained SVM models over labelled taxonomic-description sentences must be evaluated with k-fold cross validation and compared to reported precision/recall.

## When NOT to use

- The task is end-to-end phenotypic matrix extraction from new taxonomic descriptions rather than evaluating classifier accuracy (run MicroPIE normally instead)
- The text domain is not prokaryotic taxonomic descriptions; the 42-character SVM models and term lists are tailored to physiological/ecological trait sentences, not morphology-focused descriptions
- No labelled sentence instances are available, since supervised cross validation requires ground-truth labels for each sentence

## Inputs

- Labelled sentence training corpus (17,147 instances, 42 microbial phenotypic characters) from the MicroPIE 'models' package
- Term Lists defining the 42 characters, from models/CharacterValueExtractors0.1.0

## Outputs

- cv_metrics.csv with per-character and averaged 10-fold precision and recall
- Trained per-character LibSVM model files stored under models/character_model0.1.0

## How to apply

Load the labelled sentence training corpus of 17,147 instances covering 42 microbial phenotypic characters (e.g., mol %G+C, cell shape, Gram stain type, aerophilicity) from the MicroPIE 'models' package resources. Configure one LibSVM sentence classifier per character, using the Term Lists under models/CharacterValueExtractors0.1.0 to define the character set. Train each classifier on its labelled instances, then run 10-fold cross validation per classifier and record per-fold precision and recall. Average precision and recall across the 42 classifiers and write per-character and summary metrics to cv_metrics.csv, saving trained models under models/character_model0.1.0. Accept the reproduction when 10-fold cross-validation average precision is approximately 98.6% and average recall approximately 99.5%, matching the paper's reported values.

## Related tools

- **MicroPIE 0.1.0** (Source of the labelled sentence corpus, SVM character models, and Term Lists used for the cross-validation evaluation) — github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Supervised classification algorithm behind the 42 per-character LibSVM sentence classifiers being cross-validated) — github.com/biosemantics/micropie2

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- 10-fold cross-validation average precision across the 42 classifiers is approximately 98.6%
- 10-fold cross-validation average recall across the 42 classifiers is approximately 99.5%
- cv_metrics.csv contains per-character rows for all 42 phenotypic characters plus a summary row
- Every character classifier corresponds to a Term List entry in models/CharacterValueExtractors0.1.0
- Trained model files are written to models/character_model0.1.0 and loadable by MicroPIE 0.1.0

## Limitations

- Nearly half of the designated characters had Relaxed_F1 scores not above 0.80 in gold-standard evaluation, so strong character-prediction cross-validation metrics do not guarantee strong state extraction for every character
- Authors state further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types
- Metrics are specific to the 17,147-instance training corpus and 42-character scope of MicroPIE 0.1.0; version 0.2.0 adds characters and bug fixes

## Evidence

- [full_text] The trained SVMs achieved an average of 98.6% precision and 99.5% recall in character prediction, as evaluated by 10-fold cross validation on the 17,147 labelled training instances.: [evidence span withheld — anchor status: not_found]
- [discussion] SVM component rationale: "inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE"
- [abstract] SVM-based character identification from sentences: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [readme] Model and Term List locations: "The trained SVM models are in the folder, models\character_model0.1.0. The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [readme] Models folder contents: "The “models” folders contains SVM- trained model files, Term Lists and other configuration files."
