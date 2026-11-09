---
name: statistical-significance-testing-students-ttest
description: Use when comparing per-character extraction performance (Relaxed precision/recall/F1)
  of two systems—e.g., an NLP tool vs. a human-curated matrix—scored against one shared
  gold standard and a significance verdict is needed.
when_to_use_negative:
- The two systems were not evaluated against the same gold standard matrix, so the
  paired per-character scores are not comparable.
- Fewer than the 12 compared characters have complete Relaxed scores for both systems,
  undermining the paired t-Test.
- The comparison target is micro-level per-character accuracy rather than detecting
  a statistically significant aggregate performance difference between two extractors.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MicroPIE 0.1.0
  role: Generates the automated taxon-by-character matrix whose per-character Relaxed
    scores are tested against the student matrix
  repo: github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Character Predictor inside MicroPIE whose inclusion affects the scores being
    compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:2bd2e41519cec699adef7d94f542f4a20bbf824659d31696dd41aaba1df2206e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MicroPIE
  - MicroPIE 0.1.0
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# statistical-significance-testing-students-ttest

## Summary

Applies Student's t-Tests to determine whether an automated text-mining extractor (MicroPIE 0.1.0) and a human baseline (undergraduate microbiology students) differ significantly in per-character Relaxed scores when both are evaluated against the same hand-generated gold standard matrix on 46 prokaryotic taxonomic descriptions.

## When to use

Use when comparing per-character extraction performance (Relaxed precision/recall/F1) of two systems—e.g., an NLP tool vs. a human-curated matrix—scored against one shared gold standard and a significance verdict is needed.

## When NOT to use

- The two systems were not evaluated against the same gold standard matrix, so the paired per-character scores are not comparable.
- Fewer than the 12 compared characters have complete Relaxed scores for both systems, undermining the paired t-Test.
- The comparison target is micro-level per-character accuracy rather than detecting a statistically significant aggregate performance difference between two extractors.

## Inputs

- 46 prokaryotic taxonomic descriptions (input folder for MicroPIE)
- Hand-generated gold standard taxon-by-character matrix
- Taxon-by-character matrix produced by MicroPIE 0.1.0
- Student-generated taxon-by-character matrix
- Per-character Relaxed precision, recall, and F1 scores for the 12 compared characters

## Outputs

- Per-character Relaxed F1 comparison table (12 characters, MicroPIE vs. students)
- Total relaxed hit scores for both matrices (445 vs. 200)
- Student's t-Test results with significance at p < 0.01

## How to apply

Run MicroPIE 0.1.0 on the 46 taxonomic descriptions used in the student class projects, outputting a taxon-by-character matrix, and score it per character (antibiotic production, biofilm formation, cell division pattern and reproduction, film test result, filterability, host, methanogenesis products, pressure preference, spot test result, symbiotic relationship, tests negative, tests positive) against the hand-generated gold standard using Relaxed precision, recall, and F1. Compute the identical Relaxed scores for the student-generated matrix. Sum the relaxed hit scores across the 12 compared characters (MicroPIE 445 vs. students 200, a 123% difference), then apply Student's t-Tests to the paired per-character Relaxed scores and require p < 0.01 to declare the difference statistically significant. The rationale is that aggregate hit counts alone do not establish that the automated system outperformed humans character-by-character; a paired significance test over the 12 character-level scores does.

## Related tools

- **MicroPIE 0.1.0** (Generates the automated taxon-by-character matrix whose per-character Relaxed scores are tested against the student matrix) — github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Character Predictor inside MicroPIE whose inclusion affects the scores being compared)

## Examples

```
java -cp micropieproject_lib -jar micropie-0.1.0.jar -i <46_description_folder> -o <output_folder> -m <models_folder>
```

## Evaluation signals

- MicroPIE's total relaxed hit score (445) exceeds the student matrix's (200) by 123% on the 46-description subset.
- MicroPIE achieves higher Relaxed F1 than the students on all 12 compared characters against the same gold standard.
- Student's t-Test on per-character Relaxed scores yields p < 0.01.
- Both matrices are scored against the identical hand-generated gold standard, confirming a paired design.
- The 12 tested characters match the MicroPIE 0.2.0 character list (antibiotic production through tests positive) exactly.

## Limitations

- Nearly half of the designated characters had Relaxed_F1 scores not above 0.80, so significance on the 12 compared characters does not imply uniform superiority.
- Further development, including incorporation of ontologies, was judged necessary to improve extraction for some character types.
- The comparison covers only the 46 taxonomic descriptions used in the student class projects, not the full 111-description Part_One_111_final dataset.

## Evidence

- [abstract] Evaluated against the same gold standard, MicroPIE performed significantly better than the undergraduate students.: "Evaluated against the same gold standard, MicroPIE performed significantly better than the undergraduate students."
- [discussion] MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%: "MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%"
- [readme] Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [discussion] hand-generated gold standard matrix: "Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters"
