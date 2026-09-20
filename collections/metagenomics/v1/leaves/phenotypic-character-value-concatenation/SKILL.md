---
name: phenotypic-character-value-concatenation
description: Use when per-taxon extraction records from MicroPIE's character extractors
  must be pivoted into a structured taxon-by-character matrix, especially when a taxon
  has multiple extracted values for the same character that must be merged.
when_to_use_negative:
- The input is already a structured taxon-by-character matrix rather than raw natural
  language taxonomic descriptions or per-taxon extraction records
- The task requires character prediction or state extraction itself rather than matrix
  assembly — those are handled by upstream MicroPIE components (SVM Character Predictor,
  value extractors)
- Taxonomic descriptions contain little phenotypic content (e.g., morphology-only
  focus unsuited to MicroPIE's prokaryotic physiological/ecological character set)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: MicroPIE
  role: 'Provides the Matrix Generator component that pivots extracted values into
    the taxon-by-character matrix and joins multiple values with the # separator'
  repo: github.com/biosemantics/micropie2
- name: MicroPIE 0.1.0
  role: The evaluated release whose Matrix Generator assembles 42 phenotypic characters
    into the output matrix
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:2258c7897993c00fc2d3049a4f460c6ab92982f5abe1ffcb79097bafe1cd8b88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# phenotypic-character-value-concatenation

## Summary

Assembles character and character-state values extracted by MicroPIE into a final taxon-by-character matrix, concatenating multiple values per character into a single cell separated by number signs (#) so the output can be used by phylogenetic comparative analysis software.

## When to use

Use when per-taxon extraction records from MicroPIE's character extractors must be pivoted into a structured taxon-by-character matrix, especially when a taxon has multiple extracted values for the same character that must be merged.

## When NOT to use

- The input is already a structured taxon-by-character matrix rather than raw natural language taxonomic descriptions or per-taxon extraction records
- The task requires character prediction or state extraction itself rather than matrix assembly — those are handled by upstream MicroPIE components (SVM Character Predictor, value extractors)
- Taxonomic descriptions contain little phenotypic content (e.g., morphology-only focus unsuited to MicroPIE's prokaryotic physiological/ecological character set)

## Inputs

- Folder of prokaryotic taxonomic description files (passed via -i, e.g., F:/MicroPIE/datasets/Part_One_111_final)
- Per-taxon character and character-state values from MicroPIE's SVM-based Character Predictor and rule-based value extractors
- Character Term Lists and configuration files from the models folder (models\CharacterValueExtractors0.1.0)

## Outputs

- Taxon-by-character matrix file written to the output folder (-o), with one row per input taxon and one column per designated character, multiple values joined by '#'

## How to apply

Load the character and character-state values produced by the upstream rule-based extractor sub-task for each input taxonomic description. Pivot the per-taxon records so each taxon becomes one row and each of the 42 designated phenotypic characters (e.g., cell shape, NaCl optimum, aerophilicity, mol %G+C) becomes one column. Where a character has multiple extracted values for a taxon, concatenate them into a single cell joined by the number sign (#) separator. Consult the character Term Lists in the models configuration folder (models\CharacterValueExtractors0.1.0) to keep character names and value formatting consistent across columns. Write the assembled matrix to the output folder specified by the -o parameter.

## Related tools

- **MicroPIE** (Provides the Matrix Generator component that pivots extracted values into the taxon-by-character matrix and joins multiple values with the # separator) — github.com/biosemantics/micropie2
- **MicroPIE 0.1.0** (The evaluated release whose Matrix Generator assembles 42 phenotypic characters into the output matrix) — github.com/biosemantics/micropie2

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- The output file exists in the folder given by the -o parameter
- The generated matrix contains one row per input taxon in the -i input folder
- The matrix contains one column per designated character (42 in MicroPIE 0.1.0), with names consistent with the Term Lists in models\CharacterValueExtractors0.1.0
- Cells with multiple extracted values contain all values joined by '#' without loss
- The matrix structure matches the hand-generated gold standard matrix used in the evaluation

## Limitations

- Extraction quality feeding the matrix is imperfect: 21 characters (50%) achieved Relaxed F1 > 0.80 and 16 (38%) between 0.50 and 0.80 against the gold standard, so concatenated cells inherit extraction errors
- Further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types
- The output format uses '#' as an internal multi-value separator, which may need conversion for target phylogenetic analysis software such as Mesquite or MacClade

## Evidence

- [other] The Matrix Generator, the final MicroPIE component, assembles extracted values into a taxon-by-character matrix, with multiple values for a character separated by number signs (#).: [evidence span withheld — anchor status: not_found]
- [intro] character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters): "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
- [readme] -o: the output folder to place the generated taxon-by-character matrix: "-o: the output folder to place the generated taxon-by-character matrix"
- [discussion] Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters: "Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters"
