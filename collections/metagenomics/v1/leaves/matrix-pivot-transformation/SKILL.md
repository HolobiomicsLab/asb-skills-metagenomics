---
name: matrix-pivot-transformation
description: Use when per-taxon natural-language extraction records (character and
  character-state values from prokaryotic taxonomic descriptions) must be pivoted
  into a standard taxon-by-character matrix usable by phylogenetic analysis software.
when_to_use_negative:
- The input is already a structured taxon-by-character matrix and no text extraction
  is needed
- The source text is semi-structured morphological descriptions better suited to CharaParser,
  rather than prokaryotic descriptions rich in physiological and ecological traits
- The task requires extracting character types outside MicroPIE's designated 42-character
  set without extending the Term Lists
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3068
- http://edamontology.org/topic_3299
tools:
- name: MicroPIE 0.1.0
  role: Runs the extraction and Matrix Generator pipeline that pivots extracted values
    into the final taxon-by-character matrix
  repo: https://github.com/biosemantics/micropie2
- name: MicroPIE 0.2.0
  role: Updated version with bug fixes, additional characters, and integration into
    the ETC toolkit; same matrix assembly role
  repo: https://github.com/biosemantics/micropie2
- name: Mesquite
  role: Downstream phylogenetic comparative analysis software that consumes the standard
    taxon-by-character matrix format
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:358addd2869666ce509e85c74154e8110129cf3dc06e15e51da66a6cbda72192
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosemantics/micropie2
  tools:
  - Mesquite
  - MicroPIE
  - MicroPIE 0.1.0
  - MicroPIE 0.2.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3068
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# matrix-pivot-transformation

## Summary

Assembles per-taxon character and character-state values extracted by MicroPIE's extractor components into a structured taxon-by-character matrix, the standard table format (rows = taxa, columns = characters) required by phylogenetic comparative analysis software such as Mesquite and MacClade.

## When to use

Use when per-taxon natural-language extraction records (character and character-state values from prokaryotic taxonomic descriptions) must be pivoted into a standard taxon-by-character matrix usable by phylogenetic analysis software.

## When NOT to use

- The input is already a structured taxon-by-character matrix and no text extraction is needed
- The source text is semi-structured morphological descriptions better suited to CharaParser, rather than prokaryotic descriptions rich in physiological and ecological traits
- The task requires extracting character types outside MicroPIE's designated 42-character set without extending the Term Lists

## Inputs

- Per-taxon extracted character and character-state records from MicroPIE's rule-based extractors and SVM Character Predictor
- Folder of prokaryotic taxonomic description files (e.g., F:/MicroPIE/datasets/Part_One_111_final)
- MicroPIE 'models' configuration folder containing SVM model files (models\character_model0.1.0) and character Term Lists (models\CharacterValueExtractors0.1.0)

## Outputs

- Taxon-by-character matrix written to the output folder specified by the -o parameter, with multiple values per character-cell separated by number signs (#)

## How to apply

Load the extracted character and character-state values produced by the upstream extractors for each input taxon description (e.g., the folder passed via -i). Pivot the per-taxon extraction records so each taxon becomes one row and each of the 42 predefined phenotypic characters (e.g., mol %G+C, cell shape, Gram stain type, NaCl/pH/temperature optima, aerophilicity) becomes one column. Where a character has multiple extracted values for a taxon, concatenate them into a single cell joined by the number sign (#) separator. Consult the character Term Lists in the models configuration folder (models\CharacterValueExtractors0.1.0) to keep character names and value formatting consistent across columns, and write the assembled matrix to the output folder given by the -o parameter. Validation compares the generated matrix structure against a hand-generated gold standard matrix (one row per taxon, one column per designated character).

## Related tools

- **MicroPIE 0.1.0** (Runs the extraction and Matrix Generator pipeline that pivots extracted values into the final taxon-by-character matrix) — https://github.com/biosemantics/micropie2
- **MicroPIE 0.2.0** (Updated version with bug fixes, additional characters, and integration into the ETC toolkit; same matrix assembly role) — https://github.com/biosemantics/micropie2
- **Mesquite** (Downstream phylogenetic comparative analysis software that consumes the standard taxon-by-character matrix format)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- The output file exists in the output folder specified by the -o parameter
- The generated matrix has exactly one row per input taxon and one column per designated character (42 characters in MicroPIE 0.1.0)
- Multi-valued cells concatenate values with the number sign (#) separator, matching the hand-generated gold standard matrix structure
- Character names and value formatting in columns are consistent with the Term Lists in models\CharacterValueExtractors0.1.0
- Relaxed F1 scores against the hand-generated gold standard matrix are computed; MicroPIE achieved > 0.80 on 21 of 42 characters (50%) and 0.50–0.80 on 16 characters (38%)

## Limitations

- Nearly half of designated characters had Relaxed F1 scores not above 0.80; overall accuracy was 79.0%, so many columns require curation
- Further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types
- A character cell may contain multiple conflicting values (hence the # separator), reflecting extraction ambiguity rather than a single consensus state

## Evidence

- [intro] Standardize extracted character information into a structured taxon-by-character matrix format usable by phylogenetic comparative analysis software: "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
- [other] Multiple values for a character are separated by number signs in the assembled matrix: [evidence span withheld — anchor status: not_found]
- [readme] The -o parameter places the generated matrix in the output folder: "-o: the output folder to place the generated taxon-by-character matrix"
- [readme] Term Lists reside in the models folder and define character value formatting: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [discussion] Evaluation of the generated matrix against a hand-generated gold standard: "Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters"
