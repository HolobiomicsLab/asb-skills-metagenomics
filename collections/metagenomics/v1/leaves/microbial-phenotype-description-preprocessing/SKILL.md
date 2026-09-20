---
name: microbial-phenotype-description-preprocessing
description: Use when raw microbial taxonomic description text files (e.g., the Part_One_111_final
  dataset) must be segmented and cleaned before character extraction with MicroPIE
  or a similar NLP pipeline.
when_to_use_negative:
- Input text is already structured/semi-structured morphological description markup
  (CharaParser's scope), not free-text prokaryotic descriptions
- Downstream extraction does not require sentence-level parsing, so removing non-alphanumeric
  characters would destroy needed punctuation or formatting
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0219
tools:
- name: MicroPIE
  role: Pipeline whose Preprocessor stage performs sentence splitting and cleaning
    before character extraction
  repo: github.com/biosemantics/micropie2
- name: Stanford CoreNLP
  role: Sentence splitting of scientific prose in the Preprocessor stage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:838440171d547870e9e99d43bf07ce40adbced2c3818ffe2cd8429afe4e6835c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - Stanford CoreNLP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# microbial-phenotype-description-preprocessing

## Summary

Preprocessing stage of MicroPIE that segments prokaryotic taxonomic description text into sentences (Stanford CoreNLP splitting) and normalizes them by decoding XML entities and removing non-UTF-8 and non-alphanumeric characters, so downstream SVM character prediction and rule-based extraction receive clean sentences.

## When to use

Use when raw microbial taxonomic description text files (e.g., the Part_One_111_final dataset) must be segmented and cleaned before character extraction with MicroPIE or a similar NLP pipeline.

## When NOT to use

- Input text is already structured/semi-structured morphological description markup (CharaParser's scope), not free-text prokaryotic descriptions
- Downstream extraction does not require sentence-level parsing, so removing non-alphanumeric characters would destroy needed punctuation or formatting

## Inputs

- Taxonomic description text files (folder of microbial/prokaryotic descriptions, e.g., Part_One_111_final)
- XML entities and non-UTF-8 characters embedded in the description prose (removed during normalization)

## Outputs

- Clean, normalized sentences per taxon (UTF-8 alphanumeric text plus whitespace), with sentence boundaries preserved for downstream SVM character prediction

## How to apply

Read the taxonomic description files from the folder passed via MicroPIE's -i parameter (e.g., F:/MicroPIE/datasets/Part_One_111_final). Segment each description paragraph into individual sentences using Stanford CoreNLP sentence splitting on the scientific prose. In the Sentence Cleaning step, normalize each sentence by replacing predefined XML entities with their decoded characters, removing characters not compatible with the UTF-8 charset, and stripping non-alphanumeric characters so only clean alphanumeric text and whitespace remain. Write the normalized sentence stream per taxon to the designated output folder, preserving sentence boundaries because the downstream SVM Character Predictor operates on individual sentences to identify the 42 phenotypic characters before rule-based extraction.

## Related tools

- **MicroPIE** (Pipeline whose Preprocessor stage performs sentence splitting and cleaning before character extraction) — github.com/biosemantics/micropie2
- **Stanford CoreNLP** (Sentence splitting of scientific prose in the Preprocessor stage)

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Every output file exists in the designated output folder after the run
- Each extracted sentence contains only UTF-8 alphanumeric characters plus whitespace after normalization
- Sentence boundaries are preserved so each sentence can be fed individually to the SVM Character Predictor
- No XML entities or charset-incompatible characters remain in the normalized sentence stream

## Limitations

- Sentence cleaning strips non-alphanumeric characters, so punctuation-dependent signal is lost before downstream parsing
- Nearly half of the 42 designated characters still had Relaxed_F1 scores ≤ 0.80, so preprocessing alone does not guarantee high extraction performance
- Authors note future improvements (e.g., ontology incorporation) are needed to make the system more robust for some character types

## Evidence

- [methods] Sentence Cleaning normalization of taxonomic description sentences: [evidence span withheld — anchor status: not_found]
- [methods] Purpose of cleaning: reduce parsing errors downstream: [evidence span withheld — anchor status: not_found]
- [abstract] SVM operates on sentences from taxonomic descriptions: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [readme] README: input parameter is the folder holding taxonomic descriptions: "-i: the folder holding taxonomic descriptions"
- [readme] README: example dataset used to run MicroPIE: "args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models".split("\\s+");"
