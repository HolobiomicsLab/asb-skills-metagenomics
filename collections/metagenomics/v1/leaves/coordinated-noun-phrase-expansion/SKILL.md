---
name: coordinated-noun-phrase-expansion
description: Use when extracting character values from taxonomic description sentences
  where coordinated noun phrases (e.g. conjoined descriptors) may contain terms absent
  from the curated per-character Term Lists.
when_to_use_negative:
- Descriptions contain little coordination and full term coverage exists — direct
  term matching suffices without conjunct expansion
- Sentences are not noun-phrase-structured taxonomic descriptions (e.g. free narrative
  text), where the MicroPIE term-matching extraction assumptions break down
- The target task requires character identification rather than value extraction —
  MicroPIE's SVM Character Predictor is designed for that step instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3071
tools:
- name: MicroPIE
  role: Text-mining tool implementing the term-matching Character Extractor with per-character
    Term Lists and coordinated noun-phrase expansion to produce taxon-by-character
    matrices
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:4075a20e671966b039bcff550c2f1d35ebb016da9d1fc23df5c7887c727bfa7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# coordinated-noun-phrase-expansion

## Summary

A term-matching extraction strategy in MicroPIE in which, when one conjunct of a coordinated noun phrase matches a character's Term List, all conjuncts of that coordination are accepted as values for the same character, compensating for insufficient term-list coverage of prokaryotic phenotypic traits.

## When to use

Use when extracting character values from taxonomic description sentences where coordinated noun phrases (e.g. conjoined descriptors) may contain terms absent from the curated per-character Term Lists.

## When NOT to use

- Descriptions contain little coordination and full term coverage exists — direct term matching suffices without conjunct expansion
- Sentences are not noun-phrase-structured taxonomic descriptions (e.g. free narrative text), where the MicroPIE term-matching extraction assumptions break down
- The target task requires character identification rather than value extraction — MicroPIE's SVM Character Predictor is designed for that step instead

## Inputs

- Taxonomic description sentences (text files in the folder passed via -i, e.g. Part_One_111_final)
- Curated per-character Term Lists from MicroPIE models\CharacterValueExtractors0.1.0

## Outputs

- Matched character/value pairs per taxon in an intermediate character matrix
- Taxon-by-character matrix written to the folder given by the -o output parameter

## How to apply

Load the curated Term Lists for the 42 phenotypic characters from the models folder (models\CharacterValueExtractors0.1.0) in MicroPIE, then read the taxonomic description sentences supplied via the -i input folder (e.g. Part_One_111_final). Segment each description sentence into noun phrases and match each noun phrase against the per-character term lists to identify character values. Apply the coordinated noun-phrase expansion rule: when one conjunct in a coordinated noun phrase matches a term list, extract ALL conjuncts of that phrase as character values for that character, because coordination implies shared character membership even when only one term is in the list. Record matched character/value pairs per taxon in an intermediate character matrix, then write the resulting taxon-by-character matrix to the folder given by the -o parameter. Judge extraction quality against a hand-generated gold standard using Relaxed_F1, where acceptable term-matching extraction achieves Relaxed_F1 > 0.80 (achieved for 21 of 42 designated characters, 50%).

## Related tools

- **MicroPIE** (Text-mining tool implementing the term-matching Character Extractor with per-character Term Lists and coordinated noun-phrase expansion to produce taxon-by-character matrices) — github.com/biosemantics/micropie2

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Per-character Relaxed_F1 score against a hand-generated gold standard matrix exceeds 0.80 (article reports 21/42 characters, 50%, above 0.80)
- Every conjunct of a coordinated noun phrase appears in the extracted values whenever at least one conjunct matched a Term List entry
- Generated output is a taxon-by-character matrix covering the 42 MicroPIE 0.1.0 phenotypic character types (e.g. cell shape, Gram stain type, NaCl optimum, temperature range, aerophilicity)
- MicroPIE overall accuracy near 79.0% against the gold standard, with extracted values consistent with hand-curated character states

## Limitations

- Term-list coverage is incomplete: further development, including incorporation of ontologies, was deemed necessary to improve extraction for some character types
- Nearly half of the designated characters achieved Relaxed_F1 below 0.80 (16 characters between 0.50 and 0.80), so expansion is a heuristic, not a guarantee of correct value extraction
- The rule assumes coordinated conjuncts share the same character, which can propagate false positives if a coordination mixes characters

## Evidence

- [other] when one phrase in a coordinated noun phrase is recognized as a character value, all phrases in the coordination are assumed to be values for that character, overcoming insufficient term-list coverage: [evidence span withheld — anchor status: not_found]
- [readme] The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [abstract] MicroPIE extracted 21 characters (50%) with a Relaxed_F1 score > 0.80 and 16 characters (38%) with Relaxed_F1 scores ranging between 0.50 and 0.80: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [discussion] MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%: "MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%"
- [abstract] followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
