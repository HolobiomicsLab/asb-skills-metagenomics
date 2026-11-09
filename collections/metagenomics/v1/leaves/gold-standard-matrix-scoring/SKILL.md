---
name: gold-standard-matrix-scoring
description: Use when evaluating a text-mining extraction system that outputs a taxon-by-character
  matrix over a fixed character set (e.g. the 42 MicroPIE phenotypic characters) and
  a curated gold standard matrix is available.
when_to_use_negative:
- No hand-generated gold standard matrix exists for the taxonomic descriptions, so
  no per-character Relaxed_P/R/F1 can be computed
- The extraction task targets morphology-dominated semi-structured descriptions (e.g.
  plant morphological data), where CharaParser-style tools, not MicroPIE's 42 physiological/ecological
  characters, apply
- Evaluation goal is character-level with strict (exact-match) scoring rather than
  the relaxed matching metric defined by this paper
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0089
tools:
- name: MicroPIE 0.1.0
  role: Extractor whose taxon-by-character output matrix is scored per character against
    the gold standard using Relaxed_P/R/F1
  repo: github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Character Predictor component whose contribution is quantified by rescoring
    the matrix with SVM enabled vs disabled
  repo: github.com/biosemantics/micropie2
- name: Maven
  role: Build tool used to compile the MicroPIE source into the jar that generates
    the matrix under evaluation
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:e4f262249db488ca7d32e12f09d5471688dc44ec95fd6c26e95bb75f1f484b0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven
  - MicroPIE
  - MicroPIE 0.1.0
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# gold-standard-matrix-scoring

## Summary

Score an automatically generated taxon-by-character matrix against a hand-generated gold standard matrix using Relaxed_Precision, Relaxed_Recall, and Relaxed_F1 computed per character, then bin characters into performance tiers (Relaxed_F1 > 0.80; 0.50–0.80) and report overall extraction accuracy.

## When to use

Use when evaluating a text-mining extraction system that outputs a taxon-by-character matrix over a fixed character set (e.g. the 42 MicroPIE phenotypic characters) and a curated gold standard matrix is available.

## When NOT to use

- No hand-generated gold standard matrix exists for the taxonomic descriptions, so no per-character Relaxed_P/R/F1 can be computed
- The extraction task targets morphology-dominated semi-structured descriptions (e.g. plant morphological data), where CharaParser-style tools, not MicroPIE's 42 physiological/ecological characters, apply
- Evaluation goal is character-level with strict (exact-match) scoring rather than the relaxed matching metric defined by this paper

## Inputs

- Generated taxon-by-character matrix from MicroPIE (via -o output folder)
- Hand-generated gold standard taxon-by-character matrix over 42 designated characters
- Folder of 111 prokaryotic taxonomic descriptions (Part_One_111_final)

## Outputs

- Per-character Relaxed_Precision, Relaxed_Recall, and Relaxed_F1 table (e.g. relaxed_scores.csv)
- Counts of characters per performance tier (Relaxed_F1 > 0.80; 0.50–0.80)
- Overall accuracy as percentage of character values correctly extracted
- Side-by-side SVM vs no-SVM per-character comparison with overall accuracies (ablation_comparison.csv)

## How to apply

Run the extractor (MicroPIE 0.1.0, compiled with Maven into a jar, with -i the 111-description Part_One_111_final folder, -o the output matrix folder, and -m the models folder containing SVM character models and Term Lists) to generate a candidate taxon-by-character matrix. Compare this matrix against the hand-generated gold standard matrix character by character, computing Relaxed_Precision, Relaxed_Recall, and Relaxed_F1 for each of the 42 designated characters. Bin per-character Relaxed_F1 into tiers (>0.80 and 0.50–0.80) and compute overall accuracy as the percentage of character values correctly extracted (reported: 79.0%). For ablation evaluation, repeat scoring with the SVM Character Predictor disabled (sentences flow from Preprocessor directly to Character Extractors) and compare per-character Relaxed_P/R/F1 side by side; the paper's expected result is higher Relaxed_Precision for 34 of 42 characters with SVM and accuracy 79.0% versus 54.6% (3260/5969) without it.

## Related tools

- **MicroPIE 0.1.0** (Extractor whose taxon-by-character output matrix is scored per character against the gold standard using Relaxed_P/R/F1) — github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Character Predictor component whose contribution is quantified by rescoring the matrix with SVM enabled vs disabled) — github.com/biosemantics/micropie2
- **Maven** (Build tool used to compile the MicroPIE source into the jar that generates the matrix under evaluation)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Overall accuracy reproduces at about 79.0% (16/21-tier decomposition: 21 characters (50%) with Relaxed_F1 > 0.80 and 16 characters (38%) with Relaxed_F1 between 0.50 and 0.80)
- Ablation check: Relaxed_Precision is higher under the SVM configuration for 34 of the 42 characters, and no-SVM accuracy falls to about 54.6% (3260/5969 correct values, ~2000 incorrect)
- Per-character scores are computed for exactly the 42 designated characters, each with Relaxed_P, Relaxed_R, and Relaxed_F1 defined
- All extracted character values fall within the gold standard's 42-character schema (e.g. cell shape, Gram stain type, NaCl/pH/temperature minima–optima–maxima, aerophilicity)
- Output CSVs (relaxed_scores.csv, ablation_comparison.csv) contain per-character rows with both configurations scored consistently

## Limitations

- MicroPIE 0.1.0 handles only 42 character types; performance below 0.80 Relaxed_F1 for roughly half the designated characters means further development, including incorporation of ontologies, is needed for some character types
- MicroPIE targets prokaryotic taxonomic descriptions rich in physiological and ecological traits; it performed poorly on descriptions dominated by morphological data
- Evaluation is relaxed (not exact-match) scoring against a single gold standard matrix, so results are metric-dependent

## Evidence

- [abstract] MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80.: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [discussion] MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%: "MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%"
- [discussion] inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE: "inclusion of the Character Predictor component using SVM (Support Vector Machine) improved the overall performance of MicroPIE"
- [readme] Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [methods] After downloading the project, transfer it into a Maven(https://maven.apache.org/) project. You can compile the project into a .jar file.: "After downloading the project, transfer it into a <a href="https://maven.apache.org/">Maven(https://maven.apache.org/)</a> project. You can compile the project into a .jar file."
