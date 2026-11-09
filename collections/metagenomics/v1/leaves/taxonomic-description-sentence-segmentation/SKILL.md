---
name: taxonomic-description-sentence-segmentation
description: Use when input is natural-language prokaryotic taxonomic description
  text (e.g., the Part_One_111_final dataset) that must be segmented into sentences
  and normalized before phenotypic character extraction.
when_to_use_negative:
- Input is already a structured taxon-by-character matrix rather than natural-language
  taxonomic descriptions
- Input is semi-structured morphological descriptions better suited to tools like
  CharaParser — the article notes prokaryotic descriptions focus on physiological
  and ecological traits, not morphology
- Input text must retain punctuation, special symbols, or formatting, since non-alphanumeric
  characters are removed during sentence cleaning
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MicroPIE
  role: Tool whose Preprocessor stage performs sentence splitting and sentence cleaning
    on taxonomic descriptions before character extraction
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:2bad3f6132467dbe90866be837c4f83fcb205046c7317d79d5d6b5d36e09e930
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# taxonomic-description-sentence-segmentation

## Summary

Preprocessing stage of MicroPIE that splits prokaryotic taxonomic description paragraphs into individual sentences and cleans them (XML entity decoding, removal of non-UTF-8 and non-alphanumeric characters) so downstream SVM character prediction and rule-based extraction operate on clean, normalized sentences.

## When to use

Use when input is natural-language prokaryotic taxonomic description text (e.g., the Part_One_111_final dataset) that must be segmented into sentences and normalized before phenotypic character extraction.

## When NOT to use

- Input is already a structured taxon-by-character matrix rather than natural-language taxonomic descriptions
- Input is semi-structured morphological descriptions better suited to tools like CharaParser — the article notes prokaryotic descriptions focus on physiological and ecological traits, not morphology
- Input text must retain punctuation, special symbols, or formatting, since non-alphanumeric characters are removed during sentence cleaning

## Inputs

- Taxonomic description text files for prokaryotic taxa (e.g., the Part_One_111_final dataset folder passed via the -i parameter)
- MicroPIE models folder (models/ with SVM model files and Term Lists) referenced by the -m parameter

## Outputs

- Normalized sentence streams written per taxon to output text files, containing only UTF-8 alphanumeric characters plus whitespace, with sentence boundaries preserved for downstream SVM character prediction

## How to apply

Read the taxonomic description text files from the input folder (e.g., -i F:/MicroPIE/datasets/Part_One_111_final) and segment each description paragraph into individual sentences using Stanford CoreNLP sentence splitting on the scientific prose. In the Sentence Cleaning step, normalize each sentence by replacing predefined XML entities with their decoded characters, replacing characters not compatible with the UTF-8 charset, and removing non-alphanumeric characters, leaving only clean alphanumeric text and whitespace. Write the normalized sentence stream per taxon to output files in the designated output folder; sentence boundaries must be preserved because the downstream Character Predictor uses an SVM (Support Vector Machine) supervised classifier operating on individual sentences. Clean normalized sentences reduce potential syntactic parsing errors in the subsequent linguistic-rule-based extraction of characters and character states.

## Related tools

- **MicroPIE** (Tool whose Preprocessor stage performs sentence splitting and sentence cleaning on taxonomic descriptions before character extraction) — github.com/biosemantics/micropie2

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Each output file exists in the designated output folder for every input taxon
- Every extracted sentence contains only UTF-8 alphanumeric characters plus whitespace after normalization
- Sentence boundaries are preserved so downstream SVM character prediction can operate per sentence
- Normalized sentences show XML entities decoded and no non-UTF-8 or non-alphanumeric characters remain
- The pipeline runs end-to-end with the documented -i (input descriptions), -o (output folder), and -m (models) parameters

## Limitations

- Sentence cleaning removes non-alphanumeric characters, so any characters or symbols they encoded are irrecoverable after this stage
- Nearly half of the 42 designated characters had Relaxed F1 scores not above 0.80, partly reflecting upstream sentence quality limits; authors note further development (e.g., ontology incorporation) is needed and the system is not yet fully robust
- The step is specific to prokaryotic taxonomic descriptions; CharaParser-style morphology-focused pipelines performed poorly on such descriptions, so segmentation behavior may not transfer to other domains

## Evidence

- [other] In the Preprocessor's Sentence Cleaning step, sentences are normalized by replacing predefined XML entities and characters not compatible with the UTF-8 charset, and by removing non-alphanumeric characters: [evidence span withheld — anchor status: not_found]
- [other] Segment each description paragraph into individual sentences using Stanford CoreNLP sentence splitting: [evidence span withheld — anchor status: not_found]
- [abstract] uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [readme] Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
