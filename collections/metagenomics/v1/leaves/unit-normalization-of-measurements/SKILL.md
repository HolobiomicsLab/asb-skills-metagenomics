---
name: unit-normalization-of-measurements
description: Use when free-text taxonomic description sentences yield numeric measurements
  for characters like Cell Diameter, Cell Width, or Cell Length whose units vary in
  surface form and must be standardized before building a matrix.
when_to_use_negative:
- The measurements are already in a normalized numeric table or feature matrix — re-running
  extraction would re-introduce errors
- The target characters have no unit-bearing numeric states (e.g. categorical characters
  like Gram stain type), for which the keyword-window unit normalization does not
  apply
- Descriptions are morphology-poor prokaryotic texts without dimension keywords such
  as diameter, width, or length, leaving the extractor with no trigger terms
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MicroPIE 0.1.0
  role: Runs the linguistic-rule-based Character Extractor whose post-processing normalizes
    measurement units into canonical forms in the emitted taxon-by-character matrix
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:36a090efa6ce60b4e5fef53dc8b960fd63e01a0a6ef33d570ff168bc49b52d16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Unit normalization of measurements

## Summary

Post-processing step in MicroPIE's linguistic-rule-based character extractor that converts extracted numeric measurement values with heterogeneous spelled-out or symbol units (e.g. 'microns') into canonical unit forms (e.g. 'um') while preserving range notation, so cell dimension states can be written consistently into the taxon-by-character matrix.

## When to use

Use when free-text taxonomic description sentences yield numeric measurements for characters like Cell Diameter, Cell Width, or Cell Length whose units vary in surface form and must be standardized before building a matrix.

## When NOT to use

- The measurements are already in a normalized numeric table or feature matrix — re-running extraction would re-introduce errors
- The target characters have no unit-bearing numeric states (e.g. categorical characters like Gram stain type), for which the keyword-window unit normalization does not apply
- Descriptions are morphology-poor prokaryotic texts without dimension keywords such as diameter, width, or length, leaving the extractor with no trigger terms

## Inputs

- Taxonomic description documents (e.g. the Part_One_111_final dataset folder passed via -i)
- MicroPIE Term Lists of character keywords from models\CharacterValueExtractors0.1.0

## Outputs

- cell_dimension_extracts.csv containing unit-normalized cell diameter/width/length states per taxon
- Taxon-by-character matrix files in the output folder specified by -o

## How to apply

First run the rule-based extractor over sentence-split taxonomic descriptions: scan for dimension keywords (diameter, width, length) drawn from the Term Lists in models\CharacterValueExtractors0.1.0, and apply regular-expression, POS-tag, and syntactic dependency patterns that capture numeric values co-occurring with the keyword within a 3-word window. Then, in post-processing, normalize captured units to a canonical form — converting spelled-out units such as 'microns' to 'um' — while preserving range notation, so '1.2-2.4 microns' becomes '1.2-2.4 um'. Write the normalized states per taxon into cell_dimension_extracts.csv following MicroPIE's taxon-by-character matrix layout in the output folder. The rationale is that downstream phylogenetic comparative analyses (e.g. Mesquite) require a standard structured matrix, so unit heterogeneity in natural language must be collapsed to one canonical unit per character.

## Related tools

- **MicroPIE 0.1.0** (Runs the linguistic-rule-based Character Extractor whose post-processing normalizes measurement units into canonical forms in the emitted taxon-by-character matrix) — github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- cell_dimension_extracts.csv exists and is parseable as CSV
- Output contains a unit-normalized 'um' column or substring for captured measurements (e.g. '1.2-2.4 um', never 'microns')
- Range notation is preserved verbatim across normalization (both bounds retained, separator intact)
- Spot-check extracted values against the source description sentences for the diameter/width/length characters
- Rows and columns of the output matrix align with the taxon-by-character layout MicroPIE emits to the -o output folder

## Limitations

- MicroPIE performed well (Relaxed_F1 > 0.80) on only 21 of 42 designated characters, so dimension extractions feeding normalization may be incomplete
- Further development, including incorporation of ontologies, is acknowledged as necessary to improve extraction performance for some character types
- The keyword-window rule (candidate value within 3 words of the character term) can miss measurements where the keyword and value are farther apart

## Evidence

- [readme] Term Lists location: "The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [abstract] SVM + linguistic rules pipeline: "followed by a combination of algorithms applying linguistic rules with groups of known terms to extract characters as well as character states"
- [intro] matrix output requirement: [evidence span withheld — anchor status: not_found]
- [readme] output parameter: "-o: the output folder to place the generated taxon-by-character matrix"
