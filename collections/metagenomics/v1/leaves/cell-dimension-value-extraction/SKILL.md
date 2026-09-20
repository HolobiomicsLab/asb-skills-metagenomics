---
name: cell-dimension-value-extraction
description: Use when processing natural-language prokaryotic taxonomic descriptions
  to populate a taxon-by-character matrix with cell dimension (diameter, width, length)
  measurement states and units.
when_to_use_negative:
- Descriptions contain little morphological data and focus on physiological/ecological
  traits — this extractor targets morphological cell dimension statements (CharaParser
  performed poorly on such data, but dimension extraction needs morphological mentions).
- You need characters beyond Cell Diameter, Cell Width, and Cell Length — use the
  full MicroPIE pipeline covering the other of the 42 phenotypic characters instead
  of this specific extractor.
- Input is already structured (e.g., a filled taxon-by-character matrix) rather than
  natural-language text — no text extraction is needed.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MicroPIE 0.1.0
  role: Provides the shared linguistic-rule CharacterValueExtractors whose positional
    3-word-window patterns and unit normalization produce the cell dimension states
    in cell_dimension_extracts.csv
  repo: https://github.com/biosemantics/micropie2
- name: CharaParser
  role: Prior morphological-character extractor referenced as a contrast; it performed
    poorly on prokaryotic descriptions, motivating MicroPIE's rule-based approach
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:c8b6cc2fc10042988b66dc385ec0927caf8ec48208606c84123deeb2d2292dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosemantics/micropie2
  tools:
  - CharaParser
  - MicroPIE
  - MicroPIE 0.1.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cell-dimension-value-extraction

## Summary

A linguistic-rule-based value extractor from MicroPIE (v0.1.0) that identifies candidate Cell Diameter, Cell Width, and Cell Length values in prokaryotic taxonomic description sentences, using dimension keywords from Term Lists plus regex/POS/dependency patterns and unit normalization (e.g., 'microns' → 'um').

## When to use

Use when processing natural-language prokaryotic taxonomic descriptions to populate a taxon-by-character matrix with cell dimension (diameter, width, length) measurement states and units.

## When NOT to use

- Descriptions contain little morphological data and focus on physiological/ecological traits — this extractor targets morphological cell dimension statements (CharaParser performed poorly on such data, but dimension extraction needs morphological mentions).
- You need characters beyond Cell Diameter, Cell Width, and Cell Length — use the full MicroPIE pipeline covering the other of the 42 phenotypic characters instead of this specific extractor.
- Input is already structured (e.g., a filled taxon-by-character matrix) rather than natural-language text — no text extraction is needed.

## Inputs

- Folder of prokaryotic taxonomic description documents (e.g., Part_One_111_final dataset directory)
- Character Term Lists from models\CharacterValueExtractors0.1.0 (dimension keywords: diameter, width, length)
- SVM-trained model files and configuration files from the models folder (models\character_model0.1.0)

## Outputs

- cell_dimension_extracts.csv — taxon-by-character matrix rows with extracted, unit-normalized cell diameter/width/length states (e.g., '1.2-2.4 um') written to the output folder (-o)

## How to apply

Load the taxonomic description documents (e.g., the Part_One_111_final dataset directory passed via MicroPIE's -i parameter) and split them into sentences. Take the dimension character keywords (diameter, width, length) from the character Term Lists shipped in the models\CharacterValueExtractors0.1.0 folder of MicroPIE. Apply the shared rule-based extractor: regular expressions over numeric-value strings, POS tag patterns, and syntactic dependency patterns, relying on the positional rule that the character-indicating keyword follows the candidate numeric value within a 3-word window. In post-processing, normalize captured units to a canonical form — converting spelled-out units such as 'microns' to 'um' — while preserving range notation so '1.2-2.4 microns' becomes '1.2-2.4 um'. Write the extracted, normalized states per taxon into cell_dimension_extracts.csv in the taxon-by-character matrix layout MicroPIE emits to its output folder (-o).

## Related tools

- **MicroPIE 0.1.0** (Provides the shared linguistic-rule CharacterValueExtractors whose positional 3-word-window patterns and unit normalization produce the cell dimension states in cell_dimension_extracts.csv) — https://github.com/biosemantics/micropie2
- **CharaParser** (Prior morphological-character extractor referenced as a contrast; it performed poorly on prokaryotic descriptions, motivating MicroPIE's rule-based approach)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- cell_dimension_extracts.csv exists in the output folder and is parseable as CSV.
- Extracted measurement strings contain the normalized unit 'um' (e.g., '1.2-2.4 um'), confirming spelled-out units like 'microns' were standardized while range notation was preserved.
- Spot-check each extracted diameter/width/length value against its source description sentence: the numeric value must occur within 3 words before the character keyword.
- Verify the output follows the taxon-by-character matrix layout MicroPIE emits to the -o folder (rows = taxa, columns = characters).
- Benchmark against the hand-generated gold standard matrix using Relaxed_F1 — MicroPIE achieved > 0.80 on over half of designated characters (overall accuracy 79.0%).

## Limitations

- Nearly half of designated characters had Relaxed_F1 scores not above 0.80, indicating extraction performance gaps for some character types.
- The authors state further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types.
- Performance depends on the positional linguistic pattern (keyword following value within a 3-word window); sentences deviating from this pattern may be missed or mis-extracted.
- MicroPIE 0.1.0 (Apr 25, 2016) is versioned software — MicroPIE 0.2.0 includes bug fixes and extracts more characters, so results differ across versions.

## Evidence

- [abstract] Rule-based extraction with term lists and dependency patterns: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
- [intro] Taxon-by-character matrix standardization: "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
- [methods] Command-line invocation with input, output, models: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [readme] Term Lists location in models folder: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [readme] Cell diameter, length, width among 42 characters: "42 types of phenotypic characters and character states, including mol %G+C, cell shape, cell diameter, cell length, cell width"
