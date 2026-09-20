---
name: linguistic-rule-pattern-authoring
description: Use when extracting quantitative phenotypic characters (cell diameter,
  width, length) from natural-language taxonomic descriptions and a deterministic,
  term-list-driven rule pattern is preferred over supervised classification.
when_to_use_negative:
- The descriptions contain little morphological data and focus on physiological/ecological
  traits, where keyword-window rules for dimension characters will rarely fire
- A domain-independent open extraction is desired without domain knowledge — the article
  found OIE inadequate because the task requires substantial domain knowledge
- The character must be predicted where no explicit keyword co-occurs; use MicroPIE's
  SVM character prediction component instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3071
tools:
- name: MicroPIE 0.1.0
  role: Hosts the shared linguistic-rule Character Extractor, Term Lists, SVM models,
    and emits the taxon-by-character matrix
  repo: github.com/biosemantics/micropie2
- name: Stanford Parser
  role: Provides POS tags and syntactic dependencies underlying the rule patterns
    (per associated AVAToL parsing work)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:1d9c9c91f0f35c26fb3d937f12648d36b609c2a2ade25253400cd63d17ba55cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  - Stanford Parser
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# linguistic-rule-pattern-authoring

## Summary

Authoring linguistic-rule-based extraction patterns that pair character Term Lists with regex, POS-tag, and dependency patterns to pull numeric character states (e.g., cell diameter/width/length) from prokaryotic taxonomic descriptions, as implemented in MicroPIE's shared Character Extractor.

## When to use

Use when extracting quantitative phenotypic characters (cell diameter, width, length) from natural-language taxonomic descriptions and a deterministic, term-list-driven rule pattern is preferred over supervised classification.

## When NOT to use

- The descriptions contain little morphological data and focus on physiological/ecological traits, where keyword-window rules for dimension characters will rarely fire
- A domain-independent open extraction is desired without domain knowledge — the article found OIE inadequate because the task requires substantial domain knowledge
- The character must be predicted where no explicit keyword co-occurs; use MicroPIE's SVM character prediction component instead

## Inputs

- Taxonomic description documents (e.g., Part_One_111_final dataset folder of natural-language prokaryote descriptions)
- Character Term Lists from models\CharacterValueExtractors0.1.0 (keywords for diameter, width, length)

## Outputs

- cell_dimension_extracts.csv — taxon-by-character matrix of normalized cell diameter/width/length states with canonical units (um) and preserved range notation

## How to apply

Load the taxonomic description documents (e.g., the Part_One_111_final dataset folder) and split them into sentences. Draw the dimension keywords (diameter, width, length) from the Term Lists in the models\CharacterValueExtractors0.1.0 folder, then apply rule-based patterns — regular expressions over numeric-value strings, POS tag patterns, and syntactic dependency patterns — to capture measurement values co-occurring with a dimension keyword within a 3-word window, exploiting the positional pattern in which the keyword follows the candidate value. In post-processing, normalize spelled-out units to canonical forms while preserving range notation (e.g., '1.2–2.4 microns' → '1.2–2.4 μm/um'). Write extracted states per taxon into cell_dimension_extracts.csv in the taxon-by-character matrix layout MicroPIE emits. Rationale: rule extraction follows the SVM character-prediction step in MicroPIE's pipeline, and rules with known term groups were shown to reach Relaxed F1 > 0.80 on over half of the 42 designated characters.

## Related tools

- **MicroPIE 0.1.0** (Hosts the shared linguistic-rule Character Extractor, Term Lists, SVM models, and emits the taxon-by-character matrix) — github.com/biosemantics/micropie2
- **Stanford Parser** (Provides POS tags and syntactic dependencies underlying the rule patterns (per associated AVAToL parsing work))

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- cell_dimension_extracts.csv exists in the output folder and is parseable as CSV
- Normalized unit 'um' appears for captured measurements while range notation is preserved (e.g., '1.2-2.4 um')
- Spot-check extracted diameter/width/length values against the source description sentences for correctness
- Extracted states align with the taxon-by-character matrix layout (rows = taxa, columns = characters)
- Performance benchmark: Relaxed F1 > 0.80 against a hand-generated gold standard matrix for well-covered characters

## Limitations

- The 3-word positional window assumes the character keyword follows the candidate numeric value; constructions violating this order will be missed
- Nearly half of designated characters had Relaxed_F1 not above 0.80; authors state incorporation of ontologies and further development is necessary for some character types
- The system was acknowledged as not yet robust, with future improvements expected

## Evidence

- [abstract] followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
- [methods] run MicroPIE from command line with input, output, and models parameters: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [discussion] MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%: "MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%"
- [readme] Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [readme] cell diameter, cell length, cell width among the 42 handled characters: "MicroPIE 0.1.0 (released Apr 25, 2016) can cope with 42 types of phenotypic characters and character states, including mol %G+C, cell shape, cell diameter, cell length, cell width"
