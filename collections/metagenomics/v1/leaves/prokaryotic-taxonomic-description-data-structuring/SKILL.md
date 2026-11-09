---
name: prokaryotic-taxonomic-description-data-structuring
description: Use when free-text prokaryotic taxonomic descriptions must be converted
  into a structured taxon-by-character matrix (rows = taxa, columns = 42 designated
  phenotypic characters) for phylogenetic analysis.
when_to_use_negative:
- Input is already a structured taxon-by-character matrix or other tabular character
  data — no text extraction or pivoting is needed.
- 'Taxonomic descriptions are morphological or semi-structured: CharaParser performed
  poorly on prokaryotic descriptions, but this skill targets physiological/ecological
  trait text from prokaryote descriptions specifically.'
- Task requires extraction of characters outside MicroPIE's designated set (42 characters
  in v0.1.0, plus additional characters like antibiotic production and biofilm formation
  in v0.2.0).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3297
tools:
- name: MicroPIE
  role: Text-mining application whose final Matrix Generator component assembles extracted
    phenotypic characters and character states into the taxon-by-character matrix
  repo: github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Supervised classification component that identifies which characters are mentioned
    in each sentence before matrix assembly
- name: Mesquite
  role: Downstream phylogenetic comparative analysis software that consumes the standard
    structured taxon-by-character matrix format
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:66e5d4ed7d4de4a1e8307898c38f321f1b798bf5b6ba288ded4d6123c6cc2687
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mesquite
  - MicroPIE
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3297
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# prokaryotic-taxonomic-description-data-structuring

## Summary

Structures phenotypic characters and character states extracted from prokaryotic taxonomic descriptions into a standard taxon-by-character matrix usable by phylogenetic comparative analysis software, using MicroPIE's Matrix Generator to pivot per-taxon extraction records and join multiple values with the number sign (#) separator.

## When to use

Use when free-text prokaryotic taxonomic descriptions must be converted into a structured taxon-by-character matrix (rows = taxa, columns = 42 designated phenotypic characters) for phylogenetic analysis.

## When NOT to use

- Input is already a structured taxon-by-character matrix or other tabular character data — no text extraction or pivoting is needed.
- Taxonomic descriptions are morphological or semi-structured: CharaParser performed poorly on prokaryotic descriptions, but this skill targets physiological/ecological trait text from prokaryote descriptions specifically.
- Task requires extraction of characters outside MicroPIE's designated set (42 characters in v0.1.0, plus additional characters like antibiotic production and biofilm formation in v0.2.0).

## Inputs

- Folder of prokaryotic taxonomic descriptions in natural language (passed via MicroPIE -i, e.g. F:/MicroPIE/datasets/Part_One_111_final)
- MicroPIE 'models' folder containing SVM-trained model files (models\character_model0.1.0) and character Term Lists (models\CharacterValueExtractors0.1.0)
- Extracted character and character-state values from MicroPIE's upstream extractor components

## Outputs

- Taxon-by-character matrix (one row per input taxon, one column per designated character) written to the folder given by the -o parameter
- Multiple values for a single character in one cell, separated by number signs (#)

## How to apply

Load the character and character-state values produced by MicroPIE's upstream extractor components (SVM character identification followed by rule-based extraction with Term Lists) for each input taxon description. Pivot the per-taxon extraction records so each taxon becomes one row and each of the 42 predefined phenotypic characters becomes one column, consulting the character Term Lists in the models folder (models\CharacterValueExtractors0.1.0) to keep character names and value formatting consistent across columns. Where a character has multiple extracted values for a taxon, concatenate them into a single cell joined by the number sign (#) separator. Write the assembled matrix to the output folder specified by the -o parameter of the MicroPIE run. Validate against a hand-generated gold standard matrix: MicroPIE achieved Relaxed_F1 > 0.80 on 21 of 42 characters (50%) and 0.50–0.80 on 16 characters (38%), with an overall accuracy of 79.0%.

## Related tools

- **MicroPIE** (Text-mining application whose final Matrix Generator component assembles extracted phenotypic characters and character states into the taxon-by-character matrix) — github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Supervised classification component that identifies which characters are mentioned in each sentence before matrix assembly)
- **Mesquite** (Downstream phylogenetic comparative analysis software that consumes the standard structured taxon-by-character matrix format)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Output file exists in the output folder specified by the -o parameter.
- Generated matrix has exactly one row per input taxon and one column per designated character, matching the hand-generated gold standard matrix structure.
- Cells with multiple extracted values contain values joined by the number sign (#) separator.
- Character column names and value formatting are consistent with the Term Lists in models\CharacterValueExtractors0.1.0.
- Against the hand-generated gold standard, Relaxed_F1 score > 0.80 is expected on roughly half of the 42 designated characters (21/42 in the published evaluation), with overall accuracy around 79.0%.

## Limitations

- Nearly half of designated characters had Relaxed_F1 scores below 0.80 (16 characters between 0.50 and 0.80), and further development including incorporation of ontologies is necessary for some character types.
- Domain-independent approaches (OIE, DeepDive) were inadequate; the skill depends on MicroPIE's curated Term Lists and SVM models, which are version-specific (0.1.0 vs 0.2.0).
- Authors expect future improvements to make the system more robust; some character types still extract poorly.

## Evidence

- [intro] character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters): "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
- [abstract] MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80.: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [readme] The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [readme] -o: the output folder to place the generated taxon-by-character matrix: "-o: the output folder to place the generated taxon-by-character matrix"
