---
name: text-normalization-for-nlp
description: Use when raw taxonomic description files (e.g., from a folder like Part_One_111_final)
  must be segmented into sentences and cleaned before feeding NLP components such
  as an SVM character predictor.
when_to_use_negative:
- The input already consists of clean, sentence-segmented alphanumeric text and re-normalizing
  would strip punctuation or structure still needed by downstream components.
- The task requires preserving XML markup or special symbols (e.g., scientific notation
  with punctuation) in the text, since this cleaning step removes all non-alphanumeric
  characters.
- The task is extracting characters from morphological/semi-structured botanical descriptions
  rather than prokaryotic descriptions that emphasize physiological and ecological
  traits.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MicroPIE
  role: Hosts the Preprocessor stage whose Sentence Cleaning step performs XML-entity
    replacement, UTF-8 compatibility filtering, and non-alphanumeric character removal;
    run as a compiled Maven jar with -i, -o, and -m parameters
  repo: github.com/biosemantics/micropie2
- name: Stanford CoreNLP
  role: Sentence splitter used to segment description paragraphs into individual sentences
    before normalization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:04f0c387265a5d1dd8422bba1105ca91f3323e2decb49182c4fc8aa03c9a37ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - Stanford CoreNLP
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# text-normalization-for-nlp

## Summary

A preprocessing skill from the MicroPIE pipeline that splits prokaryotic taxonomic description text into sentences (Stanford CoreNLP) and normalizes each sentence by decoding XML entities and stripping non-UTF-8 and non-alphanumeric characters so downstream SVM character prediction receives clean input.

## When to use

Use when raw taxonomic description files (e.g., from a folder like Part_One_111_final) must be segmented into sentences and cleaned before feeding NLP components such as an SVM character predictor.

## When NOT to use

- The input already consists of clean, sentence-segmented alphanumeric text and re-normalizing would strip punctuation or structure still needed by downstream components.
- The task requires preserving XML markup or special symbols (e.g., scientific notation with punctuation) in the text, since this cleaning step removes all non-alphanumeric characters.
- The task is extracting characters from morphological/semi-structured botanical descriptions rather than prokaryotic descriptions that emphasize physiological and ecological traits.

## Inputs

- Plain-text taxonomic description files for prokaryotic taxa (e.g., the Part_One_111_final dataset folder passed via -i)
- MicroPIE 'models' folder with SVM-trained models, Term Lists and configuration files (passed via -m)

## Outputs

- Normalized per-taxon sentence streams written as text files in the designated output folder, containing only UTF-8 alphanumeric characters and whitespace, with sentence boundaries preserved

## How to apply

Read the taxonomic description text files from the input folder passed via the '-i' parameter (e.g., F:/MicroPIE/datasets/Part_One_111_final). Segment each description paragraph into individual sentences using Stanford CoreNLP sentence splitting on the scientific prose. In the Sentence Cleaning step, normalize each sentence by replacing predefined XML entities with their decoded characters and removing characters incompatible with the UTF-8 charset as well as non-alphanumeric characters, leaving only clean alphanumeric text and whitespace. Write the normalized sentence stream per taxon to text files in the output folder given by the '-o' parameter. The rationale is that MicroPIE's downstream Character Predictor (SVM) and rule-based term-matching components expect clean sentences, and normalization reduces potential syntactic parsing errors.

## Related tools

- **MicroPIE** (Hosts the Preprocessor stage whose Sentence Cleaning step performs XML-entity replacement, UTF-8 compatibility filtering, and non-alphanumeric character removal; run as a compiled Maven jar with -i, -o, and -m parameters) — github.com/biosemantics/micropie2
- **Stanford CoreNLP** (Sentence splitter used to segment description paragraphs into individual sentences before normalization)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Every output file exists in the designated output folder, one per input taxon.
- Each extracted sentence contains only UTF-8 alphanumeric characters plus whitespace after normalization.
- Sentence boundaries are preserved so downstream SVM character prediction receives one sentence per segment.
- XML entities are decoded (no raw entity strings remain) and no non-UTF-8 or non-alphanumeric characters survive in the output.

## Limitations

- Because the cleaning step removes all non-alphanumeric characters, any downstream component that needs punctuation or special symbols loses that information at this stage.
- The article notes nearly half of the 42 designated characters did not reach Relaxed_F1 > 0.80, and authors state future improvements are expected to make the overall system more robust, so normalization alone does not guarantee good extraction performance.

## Evidence

- [methods] Sentence Cleaning normalization of MicroPIE preprocessor: [evidence span withheld — anchor status: not_found]
- [methods] Purpose of normalization for downstream components: [evidence span withheld — anchor status: not_found]
- [readme] MicroPIE pipeline context: "utilizes domain experts' knowledge, NLP (Natural Language Processing) and machine learning techniques (Support Vector Machine) to extract phenotypic"
- [readme] Command-line parameters including input/output folders: "-i: the folder holding taxonomic descriptions; -o: the output folder to place the generated taxon-by-character matrix"
