---
name: taxonomic-description-text-mining
description: Use when you have a folder of unstructured prokaryote taxonomic description
  documents and need to populate a taxon-by-character matrix of phenotypic characters
  (e.g., cell dimensions, Gram stain, NaCl/pH/temperature optima) for downstream comparative
  analysis.
when_to_use_negative:
- Input documents are semi-structured morphological descriptions rather than prokaryotic
  taxonomic descriptions focused on physiological and ecological traits — CharaParser-style
  tools or schema-aware parsers may be more appropriate, since CharaParser performed
  poorly here for the reverse reason
- You need extraction of characters beyond the 42 supported in MicroPIE 0.1.0 (or
  the additional characters in 0.2.0) without training new models
- The task requires domain-independent open information extraction without curated
  term lists — OIE/DeepDive-style systems were found inadequate for this domain-specific
  task
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3061
- http://edamontology.org/topic_0625
tools:
- name: MicroPIE 0.1.0
  role: 'Main text-mining application: runs the 42 per-character SVM sentence classifiers
    plus rule-based state extraction to produce the taxon-by-character matrix'
  repo: https://github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Supervised classification algorithm inside MicroPIE that assigns each sentence
    in a taxonomic description to one of 42 phenotypic characters, improving precision
  repo: https://github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:1f527e3863907cc5bfecc03ba9da9c494d40e5887d9106c6cbee3d4543cb784f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosemantics/micropie2
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3061
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# taxonomic-description-text-mining

## Summary

Extract phenotypic characters and character states from natural-language prokaryotic taxonomic descriptions using MicroPIE, which combines SVM-based character prediction over sentences with linguistic rule-based extraction of characters and states, and emits a taxon-by-character matrix. It automates what is otherwise a slow manual curation task for phylogenetic comparative analysis.

## When to use

Use when you have a folder of unstructured prokaryote taxonomic description documents and need to populate a taxon-by-character matrix of phenotypic characters (e.g., cell dimensions, Gram stain, NaCl/pH/temperature optima) for downstream comparative analysis.

## When NOT to use

- Input documents are semi-structured morphological descriptions rather than prokaryotic taxonomic descriptions focused on physiological and ecological traits — CharaParser-style tools or schema-aware parsers may be more appropriate, since CharaParser performed poorly here for the reverse reason
- You need extraction of characters beyond the 42 supported in MicroPIE 0.1.0 (or the additional characters in 0.2.0) without training new models
- The task requires domain-independent open information extraction without curated term lists — OIE/DeepDive-style systems were found inadequate for this domain-specific task

## Inputs

- Folder of prokaryotic taxonomic description documents (e.g., MicroPIE datasets/Part_One_111_final)
- MicroPIE 'models' folder containing SVM-trained model files (models/character_model0.1.0), character Term Lists (models/CharacterValueExtractors0.1.0), and other configuration files

## Outputs

- Taxon-by-character matrix (rows = taxa, columns = 42 phenotypic characters) written to the output folder
- Per-character extracted and normalized character states (e.g., unit-normalized cell dimension extracts such as cell_dimension_extracts.csv)

## How to apply

Compile the biosemantics/micropie2 Maven project into a .jar file (e.g., micropie-0.1.0.jar) and run it with three parameters: -i for the folder of taxonomic descriptions (e.g., Part_One_111_final), -o for the output folder receiving the generated taxon-by-character matrix, and -m for the 'models' folder. Internally, MicroPIE first uses 42 per-character LibSVM sentence classifiers (trained models in models\character_model0.1.0; term lists in models\CharacterValueExtractors0.1.0) to identify which phenotypic character a sentence describes, then applies linguistic rule-based extractors (regular expressions over numeric strings, POS tag and dependency patterns) to capture character states; numeric dimension keywords such as diameter, width, and length are matched within a 3-word window after the candidate value, and units are normalized in post-processing (e.g., '1.2-2.4 microns' becomes '1.2-2.4 um'). Judge success by the published evaluation: Relaxed_F1 > 0.80 on 21 of 42 characters and overall accuracy of 79.0% against a hand-generated gold standard, with the SVM character-prediction component contributing higher precision.

## Related tools

- **MicroPIE 0.1.0** (Main text-mining application: runs the 42 per-character SVM sentence classifiers plus rule-based state extraction to produce the taxon-by-character matrix) — https://github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Supervised classification algorithm inside MicroPIE that assigns each sentence in a taxonomic description to one of 42 phenotypic characters, improving precision) — https://github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Generated output folder contains a parseable taxon-by-character matrix (rows = taxa, columns = 42 characters) with unit-normalized values, e.g., 'um' substrings for cell diameter/width/length instead of spelled-out 'microns'
- Against a hand-generated gold standard matrix, expect Relaxed_F1 > 0.80 on roughly 21/42 (50%) of characters and F1 between 0.50-0.80 on another 16/42 (38%), with overall accuracy near 79.0%
- The SVM character predictor reaches approximately 98.6% average precision and 99.5% average recall by 10-fold cross-validation on the 17,147 labelled sentence training instances
- Run with and without the character prediction component and confirm precision improves when the SVM component is included
- Spot-check extracted values against source description sentences for characters like Cell Diameter, Cell Width, and Cell Length (3-word positional window rule)

## Limitations

- Extraction performance is uneven: 16 of 42 characters (38%) achieved Relaxed_F1 only between 0.50 and 0.80, and further development including incorporation of ontologies is needed for some character types
- The authors state the system is not yet robust and expect future improvements to make it more effective
- Domain knowledge is baked into curated term lists and SVM models, so the pipeline is specific to prokaryotic taxonomic descriptions and the 42 (0.1.0) or 54 (0.2.0) supported characters
- MicroPIE 0.1.0 is superseded: version 0.2.0 includes bug fixes, extracts additional characters, and has been integrated into the ETC toolkit

## Evidence

- [abstract] SVM pipeline overview: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [abstract] Rule-based state extraction: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
- [results] Gold-standard performance: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [readme] Command-line invocation: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [readme] Model and term-list locations: "The trained SVM models are in the folder, models\character_model0.1.0. The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
