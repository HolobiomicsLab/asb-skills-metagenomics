---
name: xml-entity-decoding
description: Use when raw taxonomic description text (e.g., files like the Part_One_111_final
  dataset) contains XML entities or characters incompatible with UTF-8 that must be
  normalized into clean alphanumeric sentences before NLP sentence parsing and SVM
  character prediction.
when_to_use_negative:
- Input text is already clean, entity-free UTF-8 alphanumeric prose and no parsing
  errors were observed downstream
- Downstream steps require punctuation, special glyphs, or markup to be preserved
  — decoding plus alphanumeric-only stripping removes them
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MicroPIE
  role: Hosts the Preprocessor/Sentence Cleaning stage that performs XML entity decoding
    and non-UTF-8/non-alphanumeric character removal before SVM-based character prediction
  repo: github.com/biosemantics/micropie2
- name: Stanford CoreNLP
  role: Performs sentence splitting on the scientific prose so normalization is applied
    per-sentence with boundaries preserved
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:4ad9a65dad1d5abf257b769a94876d8de3e3055d1fecaf20359a0d98eef5d088
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# xml-entity-decoding

## Summary

A text-normalization step from the MicroPIE Preprocessor's Sentence Cleaning stage that decodes predefined XML entities and strips non-UTF-8 and non-alphanumeric characters from taxonomic description sentences so downstream character extraction is not derailed by syntactic parsing errors.

## When to use

Use when raw taxonomic description text (e.g., files like the Part_One_111_final dataset) contains XML entities or characters incompatible with UTF-8 that must be normalized into clean alphanumeric sentences before NLP sentence parsing and SVM character prediction.

## When NOT to use

- Input text is already clean, entity-free UTF-8 alphanumeric prose and no parsing errors were observed downstream
- Downstream steps require punctuation, special glyphs, or markup to be preserved — decoding plus alphanumeric-only stripping removes them

## Inputs

- Taxonomic description text files (e.g., the Part_One_111_final dataset folder passed via the '-i' parameter)
- Predefined list of XML entities to decode

## Outputs

- Normalized sentence stream per taxon written to output text files (output folder passed via '-o') containing only UTF-8 alphanumeric characters plus whitespace, with sentence boundaries preserved

## How to apply

Read the taxonomic description text files from the input folder (e.g., '-i F:/MicroPIE/datasets/Part_One_111_final') and segment each description paragraph into individual sentences using Stanford CoreNLP sentence splitting on the scientific prose. For each sentence, replace predefined XML entities with their decoded characters, then remove characters not compatible with the UTF-8 charset and non-alphanumeric characters, leaving only clean alphanumeric text and whitespace. Write the normalized sentence stream per taxon to the designated output folder ('-o'). The rationale is that clean, normalized sentences supplied to subsequent MicroPIE components (SVM Character Predictor and rule-based Character/State Extractor) reduce potential syntactic parsing errors.

## Related tools

- **MicroPIE** (Hosts the Preprocessor/Sentence Cleaning stage that performs XML entity decoding and non-UTF-8/non-alphanumeric character removal before SVM-based character prediction) — github.com/biosemantics/micropie2
- **Stanford CoreNLP** (Performs sentence splitting on the scientific prose so normalization is applied per-sentence with boundaries preserved)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Each expected output file exists in the output folder (one normalized sentence stream per taxon)
- Every extracted sentence contains only UTF-8 alphanumeric characters plus whitespace after normalization
- Sentence boundaries are preserved for downstream SVM character prediction
- Subsequent MicroPIE parsing components run without syntactic parsing errors attributable to entities or encoding artifacts

## Limitations

- Decoding relies on a predefined list of XML entities and characters; constructs outside that list are not covered by this step
- The authors note the overall system was not yet robust: 'We expect future improvements will make it more robust and effective.'

## Evidence

- [methods] In the Preprocessor's Sentence Cleaning step, sentences are normalized by replacing predefined XML entities and characters not compatible with the UTF-8 charset, and by removing non-alphanumeric characters: [evidence span withheld — anchor status: not_found]
- [methods] so that clean, normalized sentences are supplied to subsequent components to reduce potential syntactic parsing errors: [evidence span withheld — anchor status: not_found]
- [readme] Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft"
- [abstract] Inclusion of a character prediction component (SVM) improved the overall performance of MicroPIE, notably the precision.: "Inclusion of a character prediction component (SVM) improved the overall performance of MicroPIE, notably the precision."
