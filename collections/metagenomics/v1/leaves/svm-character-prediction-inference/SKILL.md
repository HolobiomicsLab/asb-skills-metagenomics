---
name: svm-character-prediction-inference
description: Use when extracting microbial phenotypic characters from natural-language
  prokaryote taxonomic descriptions and you need to filter which sentences are passed
  to rule-based character extractors before building a taxon-by-character matrix.
when_to_use_negative:
- Input is already a structured taxon-by-character matrix rather than free-text taxonomic
  descriptions
- Descriptions are morphological/botanical rather than prokaryotic physiological and
  ecological traits — CharaParser-style morphology extraction is the wrong template
  here
- No trained SVM models or Term Lists are available and there is no labelled corpus
  to train the 42 per-character classifiers
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: SVM (Support Vector Machine)
  role: Supervised classification algorithm used by the Character Predictor to identify
    which of the 42 phenotypic characters each sentence describes
- name: MicroPIE
  role: Host application whose pipeline places the Character Predictor between the
    Preprocessor and the rule-based Character Extractors; trained SVM models are bundled
    under models/character_model0.1.0
  repo: github.com/biosemantics/micropie2
- name: Maven
  role: Build system used to compile the MicroPIE Java sources (under 'src') into
    the micropie-0.1.0.jar that runs the SVM-predicted extraction
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:1326694a4d3ff02c339652095c78938141f51e1a2f721ded6b49d66a8ec40cac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven
  - MicroPIE
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# svm-character-prediction-inference

## Summary

A sentence-level character-prediction step from MicroPIE that routes sentences from taxonomic descriptions to one of 42 phenotypic characters using per-character LibSVM classifiers trained on 17,147 labelled instances, substantially improving extraction precision.

## When to use

Use when extracting microbial phenotypic characters from natural-language prokaryote taxonomic descriptions and you need to filter which sentences are passed to rule-based character extractors before building a taxon-by-character matrix.

## When NOT to use

- Input is already a structured taxon-by-character matrix rather than free-text taxonomic descriptions
- Descriptions are morphological/botanical rather than prokaryotic physiological and ecological traits — CharaParser-style morphology extraction is the wrong template here
- No trained SVM models or Term Lists are available and there is no labelled corpus to train the 42 per-character classifiers

## Inputs

- Natural-language prokaryotic taxonomic descriptions (e.g. the Part_One_111_final folder of 111 descriptions)
- Labelled sentence training corpus (17,147 instances, 42 phenotypic characters) from the MicroPIE models package
- SVM-trained model files (models/character_model0.1.0) and character Term Lists (models/CharacterValueExtractors0.1.0)

## Outputs

- Per-character Relaxed_Precision/Relaxed_Recall/Relaxed_F1 scores on the generated taxon-by-character matrix
- 10-fold cross-validation precision and recall per classifier, averaged across 42 classifiers (cv_metrics.csv)
- Filtered sentence-to-character assignments feeding downstream rule-based Character Extractors

## How to apply

Load the labelled sentence corpus (17,147 instances covering 42 microbial phenotypic characters) from the MicroPIE 'models' package resources and configure one LibSVM classifier per character, each paired with the Term Lists under models/CharacterValueExtractors0.1.0. Train each classifier, then validate by 10-fold cross validation, expecting ~98.6% average precision and ~99.5% average recall for character prediction. At inference, the Character Predictor (SVM) sits between the Preprocessor and the rule-based Character Extractors, forwarding only sentences predicted to describe a given character; trained model files live under models/character_model0.1.0 and are supplied to MicroPIE via the -m parameter. The ablation shows this filtering matters: without it (sentences flow directly from Preprocessor to Character Extractors), accuracy drops from 79.0% (3260/5969) to 54.6% with about 2000 incorrect values, and Relaxed_Precision is higher with SVM for 34 of 42 characters.

## Related tools

- **SVM (Support Vector Machine)** (Supervised classification algorithm used by the Character Predictor to identify which of the 42 phenotypic characters each sentence describes)
- **MicroPIE** (Host application whose pipeline places the Character Predictor between the Preprocessor and the rule-based Character Extractors; trained SVM models are bundled under models/character_model0.1.0) — github.com/biosemantics/micropie2
- **Maven** (Build system used to compile the MicroPIE Java sources (under 'src') into the micropie-0.1.0.jar that runs the SVM-predicted extraction)

## Examples

```
java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- 10-fold cross validation averages ~98.6% precision and ~99.5% recall across the 42 per-character classifiers on the 17,147 labelled instances
- Ablation: enabling the Character Predictor yields overall accuracy ~79.0% (3260/5969 correct) versus ~54.6% without SVM (~2000 incorrect values)
- Relaxed_Precision is higher with the SVM configuration for 34 of the 42 designated characters against the hand-generated gold standard matrix
- With SVM enabled, 21 characters (50%) reach Relaxed_F1 > 0.80 and 16 characters (38%) fall in 0.50–0.80 on the gold standard comparison

## Limitations

- Nearly half of the designated characters had Relaxed_F1 score not above 0.80; further development, including incorporation of ontologies, is necessary for some character types
- The authors acknowledge the system is not yet robust and expect future improvements to make it more effective
- MicroPIE 0.1.0 covers 42 character types; 12 additional characters (e.g. antibiotic production, biofilm formation) were added in MicroPIE 0.2.0

## Evidence

- [abstract] uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [readme] The trained SVM models are in the folder, models\character_model0.1.0.: "The trained SVM models are in the folder, models\character_model0.1.0."
- [readme] The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [discussion] inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE: "inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE"
- [results] The trained SVMs achieved an average of 98.6% precision and 99.5% recall in character prediction: [evidence span withheld — anchor status: not_found]
