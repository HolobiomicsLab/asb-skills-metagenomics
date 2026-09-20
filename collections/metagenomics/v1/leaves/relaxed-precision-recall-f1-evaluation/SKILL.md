---
name: relaxed-precision-recall-f1-evaluation
description: Use when a text-mining or information-extraction pipeline produces a
  character matrix (or equivalent structured extractions) that must be scored against
  a hand-curated gold standard, especially when per-character precision/recall/F1
  and component ablations are needed.
when_to_use_negative:
- When extractions must match gold standard values verbatim and no partial/relaxed
  match criterion is defensible — the relaxed scoring tolerates approximate state
  matches
- When no hand-generated gold standard matrix exists for the designated characters,
  since all metrics are computed against expert-curated ground truth
- When the target text is morphological or semi-structured descriptions better suited
  to tools like CharaParser rather than prokaryotic physiological/ecological trait
  descriptions
edam_operation: http://edamontology.org/operation_0448
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_0219
tools:
- name: MicroPIE
  role: Text-mining system whose generated taxon-by-character matrices (with and without
    the SVM Character Predictor) are the subject of the relaxed P/R/F1 evaluation
  repo: https://github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Character Predictor component; its ablation is assessed by comparing Relaxed_P/R/F1
    and overall accuracy between the two MicroPIE runs
  repo: https://github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:8fc68f055b6ece3c402cc50175dc6a644f2d6f7792f3a39927e4c34575d451eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosemantics/micropie2
  tools:
  - MicroPIE
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_0448
  edam_topics:
  - http://edamontology.org/topic_3071
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# relaxed-precision-recall-f1-evaluation

## Summary

Evaluates an automatically generated taxon-by-character matrix against a hand-generated gold standard by computing Relaxed_Precision, Relaxed_Recall, and Relaxed_F1 per character (42 phenotypic characters), plus an overall accuracy, enabling ablation comparisons such as MicroPIE with vs. without its SVM Character Predictor.

## When to use

Use when a text-mining or information-extraction pipeline produces a character matrix (or equivalent structured extractions) that must be scored against a hand-curated gold standard, especially when per-character precision/recall/F1 and component ablations are needed.

## When NOT to use

- When extractions must match gold standard values verbatim and no partial/relaxed match criterion is defensible — the relaxed scoring tolerates approximate state matches
- When no hand-generated gold standard matrix exists for the designated characters, since all metrics are computed against expert-curated ground truth
- When the target text is morphological or semi-structured descriptions better suited to tools like CharaParser rather than prokaryotic physiological/ecological trait descriptions

## Inputs

- Taxonomic description folder (e.g., Part_One_111_final) processed by the extraction system
- Generated taxon-by-character matrix (SVM-enabled run)
- Generated taxon-by-character matrix (Character-Predictor-disabled run)
- Hand-generated gold standard matrix for the 42 designated characters

## Outputs

- Per-character Relaxed_Precision, Relaxed_Recall, Relaxed_F1 for each of the 42 characters and each configuration
- Overall accuracy per configuration (e.g., 79.0% vs. 54.6%, i.e., 3260/5969 without SVM)
- Count of characters whose precision is higher in one configuration (e.g., 34/42 favoring SVM)
- ablation_comparison.csv with the side-by-side per-character comparison

## How to apply

Run the extraction system twice over the same input (e.g., MicroPIE on the Part_One_111_final folder of taxonomic descriptions, once with -m pointing at the SVM models folder and once with the Character Predictor disabled so sentences flow directly from the Preprocessor to the Character Extractors). Score each generated taxon-by-character matrix against the hand-generated gold standard by computing Relaxed_Precision, Relaxed_Recall, and Relaxed_F1 for each of the 42 designated characters. Tabulate per-character Relaxed_P/R/F1 for the two configurations side by side and compute overall accuracy (e.g., 79.0% with SVM vs. 54.6% without, where the no-SVM run yielded about 2000 incorrect values out of 5969). The relaxed matching criterion tolerates partial state matches, so it is appropriate when extracted values need not match the gold standard verbatim. Validation criterion from the paper: higher Relaxed_Precision for 34 of the 42 characters under the SVM configuration and overall accuracy of about 79.0%, confirming the Character Predictor improved performance, notably precision.

## Related tools

- **MicroPIE** (Text-mining system whose generated taxon-by-character matrices (with and without the SVM Character Predictor) are the subject of the relaxed P/R/F1 evaluation) — https://github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Character Predictor component; its ablation is assessed by comparing Relaxed_P/R/F1 and overall accuracy between the two MicroPIE runs) — https://github.com/biosemantics/micropie2

## Examples

```
Java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Per-character Relaxed_P/R/F1 values are computed for all 42 designated characters against the gold standard matrix
- Relaxed_F1 score > 0.80 is expected on over half (21/42) of designated characters for the full MicroPIE system
- The SVM-enabled configuration shows higher Relaxed_Precision for 34 of the 42 characters than the no-SVM configuration
- Overall accuracy of the SVM-enabled run is about 79.0% versus 54.6% (3260/5969) without the Character Predictor
- Results are written to ablation_comparison.csv with per-character values tabulated side by side

## Limitations

- MicroPIE achieved Relaxed_F1 > 0.80 on only about half of characters; nearly half scored below 0.80, so evaluation may reveal weak character types
- Further development, including incorporation of ontologies, is necessary to improve extraction performance for some character types
- Evaluation quality depends entirely on the hand-generated gold standard matrix, which is costly to produce

## Evidence

- [discussion] Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters: "Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters"
- [results] Without the Character Predictor, MicroPIE extracted about 2000 incorrect values, giving an accuracy of 54.6% (3260/5969): [evidence span withheld — anchor status: not_found]
- [abstract] Inclusion of a character prediction component (SVM) improved the overall performance of MicroPIE, notably the precision.: "Inclusion of a character prediction component (SVM) improved the overall performance of MicroPIE, notably the precision."
- [methods] args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models".split("\s+");: "args = "-i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models".split("\s+");"
- [intro] character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters): "character information needs to be in a standard structured format (i.e., taxon-by-character matrices, which are tables with rows for taxa and columns for characters)"
