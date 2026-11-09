---
name: term-list-based-value-extraction
description: Use when natural-language taxonomic description sentences must be converted
  into taxon-by-character values using curated term lists, especially when term-list
  coverage is insufficient for coordinated noun phrases.
when_to_use_negative:
- Descriptions are morphological/semi-structured plant-style descriptions better served
  by CharaParser — prokaryotic descriptions focus on physiological and ecological
  traits, which is MicroPIE's scope.
- Extraction requires open-domain context without domain knowledge — OIE-style systems
  were found inadequate here, but term matching alone will miss characters outside
  the 42 covered types.
- The target character types (e.g. antibiotic production, pressure preference in 0.2.0)
  are absent from the 0.1.0 Term Lists.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_0091
tools:
- name: MicroPIE
  role: Provides the term-matching Character Extractor, curated Term Lists, coordinated
    noun-phrase expansion logic, and the taxon-by-character matrix writer
  repo: github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Optional character-prediction component that identifies characters from sentences
    and improves precision of the extraction pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:a6ee1e8a9fd01ba31767fbfc9ee708f2d887b4fa7a913da87a75bac35085d752
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - SVM (Support Vector Machine)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# term-list-based-value-extraction

## Summary

Extracts phenotypic character values from prokaryotic taxonomic description sentences by segmenting them into noun phrases and matching each noun phrase against curated per-character Term Lists (models\CharacterValueExtractors0.1.0) in MicroPIE, expanding coordinated noun phrases so all conjuncts become values once one matches.

## When to use

Use when natural-language taxonomic description sentences must be converted into taxon-by-character values using curated term lists, especially when term-list coverage is insufficient for coordinated noun phrases.

## When NOT to use

- Descriptions are morphological/semi-structured plant-style descriptions better served by CharaParser — prokaryotic descriptions focus on physiological and ecological traits, which is MicroPIE's scope.
- Extraction requires open-domain context without domain knowledge — OIE-style systems were found inadequate here, but term matching alone will miss characters outside the 42 covered types.
- The target character types (e.g. antibiotic production, pressure preference in 0.2.0) are absent from the 0.1.0 Term Lists.

## Inputs

- Taxonomic description sentences (folder of text files, e.g. Part_One_111_final)
- Curated Term Lists per character from models\CharacterValueExtractors0.1.0
- MicroPIE models folder passed via -m

## Outputs

- Matched character/value pairs per taxon in an intermediate character matrix
- Taxon-by-character matrix written to the -o output folder

## How to apply

Load the curated Term Lists for the 42 phenotypic characters from the models folder (models\CharacterValueExtractors0.1.0) shipped with MicroPIE. Read taxonomic description sentences from the input folder given by the -i parameter (e.g. Part_One_111_final), segment them into noun phrases, and match each noun phrase against the per-character term lists to identify character values. Apply coordinated noun-phrase expansion: when one conjunct in a coordinated noun phrase matches a term list, extract all conjuncts of that phrase as values for that character — this compensates for insufficient term-list coverage. Record matched character/value pairs per taxon in an intermediate character matrix and write the taxon-by-character matrix to the folder given by -o. Term matching is the rule-based stage of MicroPIE and can be complemented by the SVM character predictor, which notably improves precision.

## Related tools

- **MicroPIE** (Provides the term-matching Character Extractor, curated Term Lists, coordinated noun-phrase expansion logic, and the taxon-by-character matrix writer) — github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Optional character-prediction component that identifies characters from sentences and improves precision of the extraction pipeline)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Relaxed_F1 score > 0.80 against a hand-generated gold standard matrix for the majority of designated characters (achieved for 21 of 42 characters).
- Every matched value's character corresponds to a Term List entry in models\CharacterValueExtractors0.1.0.
- All conjuncts of a coordinated noun phrase are emitted together as values whenever a single conjunct matched.
- Output is a taxon-by-character matrix with taxa as rows and characters (e.g. cell shape, Gram stain type, NaCl optimum) as columns.
- Overall accuracy around 79.0% relative to the gold standard is expected; substantially lower suggests term lists or segmentation are misconfigured.

## Limitations

- Nearly half of designated characters had Relaxed_F1 not above 0.80 (16 characters scored 0.50–0.80), so term matching alone is insufficient for some character types.
- Further development, including incorporation of ontologies, is necessary to improve extraction for some character types.
- Version 0.1.0 covers only 42 character types; additional characters (e.g. antibiotic production, biofilm formation) require MicroPIE 0.2.0.

## Evidence

- [readme] Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [abstract] followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
- [methods] Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [abstract] MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80.: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [abstract] further development, including incorporation of ontologies, will be necessary to improve the performance of the extraction for some character types: "However, further development, including incorporation of ontologies, will be necessary to improve the performance of the extraction for some character types."
