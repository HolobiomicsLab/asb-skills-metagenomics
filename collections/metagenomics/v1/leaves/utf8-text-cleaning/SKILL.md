---
name: utf8-text-cleaning
description: Use when raw natural-language taxonomic or scientific prose contains
  XML entities, non-UTF-8 symbols, or punctuation that would break sentence splitting
  or downstream NLP parsing before character extraction.
when_to_use_negative:
- Input is already clean, normalized plain text with no XML entities or non-UTF-8
  characters
- Input is structured or semi-structured markup where punctuation and symbols carry
  extraction meaning (stripping non-alphanumerics would destroy it)
- The task targets morphological characters in semi-structured descriptions better
  suited to a tool like CharaParser rather than prokaryotic physiological/ecological
  prose
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MicroPIE
  role: Hosts the Preprocessor stage whose Sentence Cleaning step performs XML-entity
    replacement and removal of non-UTF-8 and non-alphanumeric characters
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:5d1c20a4c40934302e1ba9a4a4910da52660a8349bf7ec15709ae7a4388538b6
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

# utf8-text-cleaning

## Summary

A preprocessing step from the MicroPIE pipeline that segments taxonomic description paragraphs into sentences and normalizes them by decoding XML entities and stripping non-UTF-8 and non-alphanumeric characters, so downstream SVM character prediction and extraction receive clean text with fewer syntactic parsing errors.

## When to use

Use when raw natural-language taxonomic or scientific prose contains XML entities, non-UTF-8 symbols, or punctuation that would break sentence splitting or downstream NLP parsing before character extraction.

## When NOT to use

- Input is already clean, normalized plain text with no XML entities or non-UTF-8 characters
- Input is structured or semi-structured markup where punctuation and symbols carry extraction meaning (stripping non-alphanumerics would destroy it)
- The task targets morphological characters in semi-structured descriptions better suited to a tool like CharaParser rather than prokaryotic physiological/ecological prose

## Inputs

- Taxonomic description text files from an input folder holding prokaryotic descriptions (e.g., the Part_One_111_final dataset)
- Predefined mapping of XML entities to their decoded characters

## Outputs

- Normalized sentence streams written per taxon to text files in the output folder, containing only UTF-8 alphanumeric characters and whitespace with sentence boundaries preserved

## How to apply

Read the taxonomic description text files from the input folder (e.g., F:/MicroPIE/datasets/Part_One_111_final) and pass them to the MicroPIE Preprocessor stage. First segment each description paragraph into individual sentences using Stanford CoreNLP sentence splitting on the scientific prose. Then, in the Sentence Cleaning step, normalize each sentence by replacing predefined XML entities with their decoded characters and removing characters incompatible with the UTF-8 charset as well as all non-alphanumeric characters, leaving only alphanumeric text and whitespace. Write the normalized sentence stream per taxon to text files in the designated output folder, preserving sentence boundaries so each clean sentence can be fed to the SVM character predictor.

## Related tools

- **MicroPIE** (Hosts the Preprocessor stage whose Sentence Cleaning step performs XML-entity replacement and removal of non-UTF-8 and non-alphanumeric characters) — github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Each output file exists in the designated output folder for every input taxon
- Every extracted sentence contains only UTF-8 alphanumeric characters plus whitespace after normalization
- Sentence boundaries are preserved so sentences remain intact for downstream SVM character prediction
- No XML entities (e.g., &amp;-style codes) remain in the normalized sentence stream
- Parsing of subsequent components proceeds without syntactic errors attributable to dirty characters

## Limitations

- Removing non-alphanumeric characters strips punctuation, so the output is only suitable for components expecting alphanumeric-and-whitespace text
- Nearly half of designated characters had Relaxed_F1 scores not above 0.80, and authors note future improvements (including ontologies) are needed — cleaning alone does not guarantee high extraction performance

## Evidence

- [methods] sentences are normalized by replacing predefined XML entities and characters not compatible with the UTF-8 charset, and by removing non-alphanumeric characters: [evidence span withheld — anchor status: not_found]
- [discussion] so that clean, normalized sentences are supplied to subsequent components to reduce potential syntactic parsing errors: [evidence span withheld — anchor status: not_found]
- [readme] args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models".split("\s+");: "args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models".split("\\s+");"
