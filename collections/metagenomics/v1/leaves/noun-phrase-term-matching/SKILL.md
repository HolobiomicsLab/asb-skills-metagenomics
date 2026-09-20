---
name: noun-phrase-term-matching
description: Use when extracting phenotypic character values (e.g., cell shape, Gram
  stain type, NaCl/pH/temperature optima, aerophilicity) from natural-language taxonomic
  description sentences of prokaryotes without training data for supervised classification.
when_to_use_negative:
- The task requires morphology-focused extraction from semi-structured descriptions
  better handled by CharaParser, since prokaryotic descriptions contain little morphological
  data and focus on physiological and ecological traits
- You need domain-independent open extraction without domain knowledge; OIE was shown
  inadequate because this task required substantial domain knowledge
- A trained SVM character model is available and character prediction is desired to
  improve precision — use MicroPIE's full pipeline rather than pure term matching
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_0091
tools:
- name: MicroPIE 0.1.0
  role: Provides the Character Extractor whose term-matching mechanism uses the curated
    Term Lists (models\CharacterValueExtractors0.1.0) to identify character values
    from noun phrases in taxonomic descriptions
  repo: github.com/biosemantics/micropie2
- name: MicroPIE
  role: The application run from the command line with -i, -o, and -m parameters to
    execute the extraction and write the taxon-by-character matrix
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:cfdf75b7c805998c23067a5fd3f47823c56cf63b7cec9494da1984cfd231945f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# noun-phrase-term-matching

## Summary

A term-matching character extraction mechanism from MicroPIE that segments prokaryotic taxonomic description sentences into noun phrases and matches them against curated per-character Term Lists (models\CharacterValueExtractors0.1.0) to populate a taxon-by-character matrix, using coordinated noun-phrase expansion to compensate for insufficient term-list coverage.

## When to use

Use when extracting phenotypic character values (e.g., cell shape, Gram stain type, NaCl/pH/temperature optima, aerophilicity) from natural-language taxonomic description sentences of prokaryotes without training data for supervised classification.

## When NOT to use

- The task requires morphology-focused extraction from semi-structured descriptions better handled by CharaParser, since prokaryotic descriptions contain little morphological data and focus on physiological and ecological traits
- You need domain-independent open extraction without domain knowledge; OIE was shown inadequate because this task required substantial domain knowledge
- A trained SVM character model is available and character prediction is desired to improve precision — use MicroPIE's full pipeline rather than pure term matching

## Inputs

- Taxonomic description sentences for prokaryotic taxa (plain-text files in the -i input folder, e.g., F:/MicroPIE/datasets/Part_One_111_final)
- Curated per-character Term Lists from models\CharacterValueExtractors0.1.0 (shipped in the MicroPIE models folder)

## Outputs

- Matched character/value pairs per taxon in an intermediate character matrix
- Taxon-by-character matrix written to the -o output folder (e.g., F:/MicroPIE/ext/craft)

## How to apply

Load the curated Term Lists for the 42 phenotypic characters from the models\CharacterValueExtractors0.1.0 folder of MicroPIE (distinct from the SVM models in models\character_model0.1.0). Read taxonomic description sentences from the input folder given by the -i parameter, segment them into noun phrases, and match each noun phrase against the per-character term lists. Apply the coordinated noun-phrase expansion rule: when one conjunct of a coordinated noun phrase matches a term list, extract ALL conjuncts of that phrase as character values — this compensates for incomplete term-list coverage. Record matched character/value pairs per taxon in an intermediate character matrix, then write the final taxon-by-character matrix to the folder given by -o. Judge correctness against a hand-generated gold standard using Relaxed_F1, where acceptable term-matching extraction achieves Relaxed_F1 > 0.80 (MicroPIE reached this on 21 of 42 characters, i.e., 50%).

## Related tools

- **MicroPIE 0.1.0** (Provides the Character Extractor whose term-matching mechanism uses the curated Term Lists (models\CharacterValueExtractors0.1.0) to identify character values from noun phrases in taxonomic descriptions) — github.com/biosemantics/micropie2
- **MicroPIE** (The application run from the command line with -i, -o, and -m parameters to execute the extraction and write the taxon-by-character matrix) — github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Extracted character values match a hand-generated gold standard matrix with Relaxed_F1 > 0.80 for the designated characters (achievable for the majority, e.g., 21 of 42 characters at 50%)
- Coordinated noun-phrase expansion fires: when one conjunct matches a term list, all conjuncts are extracted as values for that character
- The output folder (-o) contains a valid taxon-by-character matrix with taxa as rows and the 42 phenotypic characters as columns
- Overall extraction accuracy against the gold standard is approximately 79.0% as reported for MicroPIE

## Limitations

- Term-list coverage is insufficient for some characters; coordinated noun-phrase expansion is a heuristic that can over-extract when unrelated conjuncts share coordination with a matching term
- Only 21 of 42 characters (50%) achieved Relaxed_F1 > 0.80; 16 characters (38%) fell between 0.50 and 0.80, so further development including incorporation of ontologies is necessary
- MicroPIE 0.1.0 is limited to 42 character types; additional characters (antibiotic production, biofilm formation, etc.) only appear in version 0.2.0

## Evidence

- [abstract] followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
- [methods] args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models": "args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models""
- [discussion] MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%: "MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%"
- [readme] The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
