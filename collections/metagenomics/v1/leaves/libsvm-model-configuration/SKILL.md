---
name: libsvm-model-configuration
description: 'Use when reproducing or adapting MicroPIE''s Character Predictor: you
  have labelled sentences over 42 microbial phenotypic characters and need to train/configure
  the per-character LibSVM models from the MicroPIE ''models'' resources.'
when_to_use_negative:
- Your task is extracting character states (values) rather than predicting which character
  a sentence describes — that downstream step uses linguistic rules with groups of
  known terms, not the SVM classifiers.
- Your input descriptions are morphological rather than prokaryotic physiological/ecological
  — the paper notes CharaParser-style approaches fit morphology, while these 42 SVM
  characters target prokaryote traits.
- You need a general-purpose text classifier outside MicroPIE's 42-character taxonomy;
  the Term Lists and trained models are specific to that character set.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3071
tools:
- name: MicroPIE
  role: Host application whose 'models' package supplies the labelled training instances,
    Term Lists, and the models/character_model0.1.0 destination for the trained LibSVM
    classifiers
  repo: https://github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Supervised classification algorithm used to train one sentence classifier
    per phenotypic character and evaluated by 10-fold cross validation
  repo: https://github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:e655af620022f68e8f613dadd2e6a2abfb82e2536573265bbdfb2364ea0cbe12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosemantics/micropie2
  tools:
  - MicroPIE
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# libsvm-model-configuration

## Summary

Configure and train 42 per-character LibSVM sentence classifiers for MicroPIE's Character Predictor, which assigns sentences from prokaryotic taxonomic descriptions to one of 42 phenotypic characters, and validate them by 10-fold cross validation (≈98.6% precision, ≈99.5% recall on 17,147 labelled instances).

## When to use

Use when reproducing or adapting MicroPIE's Character Predictor: you have labelled sentences over 42 microbial phenotypic characters and need to train/configure the per-character LibSVM models from the MicroPIE 'models' resources.

## When NOT to use

- Your task is extracting character states (values) rather than predicting which character a sentence describes — that downstream step uses linguistic rules with groups of known terms, not the SVM classifiers.
- Your input descriptions are morphological rather than prokaryotic physiological/ecological — the paper notes CharaParser-style approaches fit morphology, while these 42 SVM characters target prokaryote traits.
- You need a general-purpose text classifier outside MicroPIE's 42-character taxonomy; the Term Lists and trained models are specific to that character set.

## Inputs

- Labelled sentence training corpus of 17,147 instances covering 42 microbial phenotypic characters (from MicroPIE 'models' package resources)
- Term Lists defining the 42 characters (models/CharacterValueExtractors0.1.0)

## Outputs

- 42 trained per-character LibSVM model files stored under models/character_model0.1.0
- cv_metrics.csv with per-character and summary 10-fold cross-validation precision and recall

## How to apply

Load the labelled sentence training corpus (17,147 instances covering 42 microbial phenotypic characters, e.g. cell shape, Gram stain type, motility, NaCl/pH/temperature minima-optima-maxima) from the MicroPIE 'models' package resources. Configure one LibSVM sentence classifier per character, using the character Term Lists found under models/CharacterValueExtractors0.1.0 to define the 42 classes. Train each classifier on its labelled sentence instances, then run 10-fold cross validation per classifier and record per-fold precision and recall for character prediction. Average precision and recall across the 42 classifiers, write per-character and summary metrics to cv_metrics.csv, and store the trained models under models/character_model0.1.0. Accept the configuration as correct when cross-validated average precision is approximately 98.6% and average recall approximately 99.5%, matching the reported Character Predictor performance. Note that in MicroPIE the SVM character prediction step precedes rule-based extraction of character states and measurably improves overall precision.

## Related tools

- **MicroPIE** (Host application whose 'models' package supplies the labelled training instances, Term Lists, and the models/character_model0.1.0 destination for the trained LibSVM classifiers) — https://github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Supervised classification algorithm used to train one sentence classifier per phenotypic character and evaluated by 10-fold cross validation) — https://github.com/biosemantics/micropie2

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- 10-fold cross-validation average precision across the 42 classifiers is approximately 98.6%.
- 10-fold cross-validation average recall across the 42 classifiers is approximately 99.5%.
- cv_metrics.csv contains per-character precision/recall rows for all 42 characters plus summary averages.
- Trained model files are present under models/character_model0.1.0 and Term Lists under models/CharacterValueExtractors0.1.0 cover all 42 characters.
- The project compiles via Maven into a micropie jar and the Character Predictor runs end-to-end with -i/-o/-m parameters.

## Limitations

- Nearly half of designated characters had Relaxed_F1 scores not above 0.80 in overall MicroPIE extraction; the SVM component improves precision but further development, including ontology incorporation, was deemed necessary for some character types.
- MicroPIE 0.1.0 covers exactly 42 phenotypic characters; characters outside that list (e.g. antibiotic production, biofilm formation) require MicroPIE 0.2.0 models.
- Validation depends on the fixed 17,147-instance labelled corpus; results may not transfer to unlabelled or differently distributed taxonomic descriptions.

## Evidence

- [other] The trained SVMs achieved an average of 98.6% precision and 99.5% recall in character prediction, as evaluated by 10-fold cross validation on the 17,147 labelled training instances.: [evidence span withheld — anchor status: not_found]
- [abstract] uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [discussion] inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE: "inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE"
- [readme] The trained SVM models are in the folder, models\character_model0.1.0.: "The trained SVM models are in the folder, models\character_model0.1.0."
- [readme] The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
