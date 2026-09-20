---
name: term-list-driven-output-formatting
description: Use when assembling per-taxon extraction records into a structured phenotypic
  character matrix for phylogenetic analysis software such as Mesquite or MacClade.
when_to_use_negative:
- Input is already a structured taxon-by-character matrix and needs no re-extraction
  or re-formatting
- Descriptions are semi-structured morphological descriptions better suited to CharaParser,
  since prokaryotic descriptions contain little morphological data
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: MicroPIE
  role: Runs the extraction pipeline whose final Matrix Generator assembles the taxon-by-character
    matrix; Term Lists and SVM models are read from the -m models folder
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:9ccc614528590265776966c3f8416a1cf55ba47b2262c134ea24454ab38bee08
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Term-list-driven output formatting

## Summary

Formats MicroPIE's extracted character and character-state values into a taxon-by-character matrix whose character names and value formatting are kept consistent by consulting the Term Lists in the models configuration folder, with multiple values per character joined by the number sign (#).

## When to use

Use when assembling per-taxon extraction records into a structured phenotypic character matrix for phylogenetic analysis software such as Mesquite or MacClade.

## When NOT to use

- Input is already a structured taxon-by-character matrix and needs no re-extraction or re-formatting
- Descriptions are semi-structured morphological descriptions better suited to CharaParser, since prokaryotic descriptions contain little morphological data

## Inputs

- Per-taxon extracted character and character-state values produced by MicroPIE's character extractors
- Character Term Lists from the models configuration folder (models\CharacterValueExtractors0.1.0)
- Folder of prokaryotic taxonomic descriptions (e.g. Part_One_111_final)

## Outputs

- Taxon-by-character matrix (one row per input taxon, one column per designated character) written to the output folder specified by -o, with multiple values per cell separated by #

## How to apply

Load the extracted character and character-state values that the upstream MicroPIE extractors produced for each input taxon description, then pivot the per-taxon records so each taxon is one row and each of the 42 designated phenotypic characters is one column. Where a character has multiple extracted values for a taxon, concatenate them into a single cell joined by the number sign (#) separator. Consult the character Term Lists in the models folder (models\CharacterValueExtractors0.1.0) to keep character names and value formatting consistent across columns. Write the assembled taxon-by-character matrix to the output folder passed via the -o parameter; validate against the hand-generated gold standard matrix structure.

## Related tools

- **MicroPIE** (Runs the extraction pipeline whose final Matrix Generator assembles the taxon-by-character matrix; Term Lists and SVM models are read from the -m models folder) — github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Output file exists in the folder specified by the -o parameter
- Matrix has exactly one row per input taxon description
- Matrix has one column per designated character (42 characters in MicroPIE 0.1.0)
- Multiple values for a character within one cell are consistently joined by the number sign (#)
- Character names and value formatting match the Term Lists and the hand-generated gold standard matrix structure

## Limitations

- MicroPIE performed well (Relaxed_F1 > 0.80) on only about half of designated characters, so some extracted values fed into the matrix may be unreliable
- Further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types
- No changelog found in the repository; behavior differences between MicroPIE 0.1.0 and 0.2.0 must be checked manually

## Evidence

- [other] The Matrix Generator, the final MicroPIE component, assembles extracted values into a taxon-by-character matrix, with multiple values for a character separated by number signs (#).: [evidence span withheld — anchor status: not_found]
- [readme] The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [intro] character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters): "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
- [readme] -o: the output folder to place the generated taxon-by-character matrix: "-o: the output folder to place the generated taxon-by-character matrix"
