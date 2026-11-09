---
name: keyword-window-value-matching
description: Use when extracting numeric measurement values (with ranges and units)
  for dimension characters like cell diameter, width, or length from natural-language
  prokaryotic taxonomic description sentences.
when_to_use_negative:
- The text is unstructured prose without predictable keyword-adjacent numeric patterns;
  MicroPIE's rule-based extractor assumes taxonomic description sentences where keywords
  follow values
- The target characters are non-dimension qualitative traits (e.g., Gram stain type,
  aerophilicity) better handled by MicroPIE's SVM character predictor than keyword-window
  value matching
- Descriptions are morphology-poor prokaryotic texts where CharaParser-style morphological
  extraction is expected to fail — this skill targets dimension keywords, not general
  morphology
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3297
tools:
- name: MicroPIE 0.1.0
  role: Provides the shared linguistic-rule Character Value Extractor, Term Lists,
    and SVM models that implement the 3-word keyword-window value matching and unit
    normalization
  repo: github.com/biosemantics/micropie2
- name: MicroPIE 0.2.0
  role: Updated release with bug fixes and additional characters, integrated into
    the ETC toolkit
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:381ecaeaec3b6bf41268eec4b233bfa297ca967967d8a97b9f75547841a89ac7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  - MicroPIE 0.2.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3297
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# keyword-window-value-matching

## Summary

A linguistic-rule-based value extraction technique from MicroPIE that identifies candidate numeric measurement values for characters such as Cell Diameter, Cell Width, and Cell Length by matching character-indicating keywords within a 3-word window after the value, followed by unit normalization (e.g., 'microns' → 'μm'/'um').

## When to use

Use when extracting numeric measurement values (with ranges and units) for dimension characters like cell diameter, width, or length from natural-language prokaryotic taxonomic description sentences.

## When NOT to use

- The text is unstructured prose without predictable keyword-adjacent numeric patterns; MicroPIE's rule-based extractor assumes taxonomic description sentences where keywords follow values
- The target characters are non-dimension qualitative traits (e.g., Gram stain type, aerophilicity) better handled by MicroPIE's SVM character predictor than keyword-window value matching
- Descriptions are morphology-poor prokaryotic texts where CharaParser-style morphological extraction is expected to fail — this skill targets dimension keywords, not general morphology

## Inputs

- Folder of taxonomic description documents (e.g., F:/MicroPIE/datasets/Part_One_111_final)
- Character Term Lists from MicroPIE models\CharacterValueExtractors0.1.0 (keywords for diameter, width, length)

## Outputs

- cell_dimension_extracts.csv — taxon-by-character matrix of normalized cell diameter/width/length states with canonical units (e.g., '1.2-2.4 um')

## How to apply

Load the taxonomic description documents (e.g., the Part_One_111_final dataset folder) and split them into sentences. Scan each sentence for dimension keywords (diameter, width, length) drawn from the character Term Lists in the models\CharacterValueExtractors0.1.0 folder of MicroPIE. Apply rule-based extraction patterns — regular expressions over numeric-value strings, POS tag patterns, and syntactic dependency patterns — to capture measurement values that co-occur with a dimension keyword within a 3-word window, where the keyword FOLLOWS the candidate value. In post-processing, normalize captured units to canonical form, converting spelled-out units such as 'microns' to 'um' while preserving range notation (e.g., '1.2-2.4 microns' becomes '1.2-2.4 um'). Write the extracted and normalized states per taxon into cell_dimension_extracts.csv in the taxon-by-character matrix layout MicroPIE emits to the output folder.

## Related tools

- **MicroPIE 0.1.0** (Provides the shared linguistic-rule Character Value Extractor, Term Lists, and SVM models that implement the 3-word keyword-window value matching and unit normalization) — github.com/biosemantics/micropie2
- **MicroPIE 0.2.0** (Updated release with bug fixes and additional characters, integrated into the ETC toolkit) — github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- cell_dimension_extracts.csv exists and parses as valid CSV in the MicroPIE output folder
- Extracted measurement strings contain the normalized unit 'um' (e.g., '1.2-2.4 um') rather than spelled-out units like 'microns'
- Range notation is preserved after normalization (en-dash/hyphen between min and max values)
- Spot-check extracted diameter/width/length values against the source description sentences to confirm the keyword appears within 3 words after the value
- Extracted states align with the taxon-by-character matrix layout MicroPIE emits (rows for taxa, columns for characters)

## Limitations

- MicroPIE achieved Relaxed_F1 > 0.80 on only 21 of 42 designated characters; nearly half of characters performed below that threshold, so dimension extraction may miss values
- Rule-based keyword-window matching depends on predictable sentence positions; it can fail on reordered phrasings where the keyword precedes the value or is separated by more than 3 words
- Authors note further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types

## Evidence

- [abstract] followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states: [evidence span withheld — anchor status: not_found]
- [abstract] uses supervised classification and linguistic rules: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [readme] Term Lists of characters are in models\CharacterValueExtractors0.1.0: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [methods] run MicroPIE with input, output, and models parameters: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [intro] character information needs a standard structured taxon-by-character matrix format: "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
