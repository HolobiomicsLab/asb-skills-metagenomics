---
name: information-extraction-performance-evaluation
description: Use when an NLP-based extractor produces structured outputs (e.g., taxon-by-character
  matrices) from text and you must quantify its accuracy against a manually curated
  gold standard or compare it with human annotators.
when_to_use_negative:
- Input texts are not natural-language taxonomic descriptions but already structured
  matrices or databases
- The target characters are morphological rather than the physiological/ecological
  traits MicroPIE was designed for (CharaParser's poor performance domain)
- No hand-generated gold standard matrix exists or can be curated for the evaluation
  set
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3071
tools:
- name: MicroPIE 0.1.0
  role: Text-mining extractor whose SVM-based character prediction and rule-based
    extraction outputs are being evaluated; run from the command line with -i (descriptions
    folder), -o (output folder), -m (models folder)
  repo: github.com/biosemantics/micropie2
- name: Maven
  role: Build system used to compile the MicroPIE project into a micropie-0.1.0.jar
    file before evaluation
- name: SVM (Support Vector Machine)
  role: Supervised classification component that identifies characters from sentences;
    its contribution to precision is measured by comparing performance with and without
    it
  repo: github.com/biosemantics/micropie2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:5318472c5b89602c26f1b095437e98e5426300e43d11c532459caedeccd8e0d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven
  - MicroPIE
  - MicroPIE 0.1.0
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# information-extraction-performance-evaluation

## Summary

Evaluates a text-mining information extraction system (e.g., MicroPIE) against a hand-generated gold standard taxon-by-character matrix using Relaxed precision, recall, and F1, and benchmarks it against human (undergraduate student) extraction with Student's t-Tests.

## When to use

Use when an NLP-based extractor produces structured outputs (e.g., taxon-by-character matrices) from text and you must quantify its accuracy against a manually curated gold standard or compare it with human annotators.

## When NOT to use

- Input texts are not natural-language taxonomic descriptions but already structured matrices or databases
- The target characters are morphological rather than the physiological/ecological traits MicroPIE was designed for (CharaParser's poor performance domain)
- No hand-generated gold standard matrix exists or can be curated for the evaluation set

## Inputs

- Corpus of natural-language prokaryotic taxonomic descriptions (e.g., 46-description subset or Part_One_111_final dataset)
- Extracted taxon-by-character matrix produced by the IE system (e.g., MicroPIE output)
- Hand-generated gold standard taxon-by-character matrix
- Optional: human-produced (student) taxon-by-character matrix as baseline

## Outputs

- Per-character Relaxed precision, recall, and F1 scores
- Total relaxed hit score comparison (extractor vs. human baseline)
- Student's t-Test results over per-character scores with p-values

## How to apply

Run the extractor (e.g., MicroPIE 0.1.0 via 'java -cp micropieproject_lib -jar micropie-0.1.0.jar -i <descriptions> -o <output> -m <models>') on the target corpus of taxonomic descriptions and collect the generated taxon-by-character matrix. Build or reuse a hand-generated gold standard matrix over the same input set (e.g., 46 prokaryotic descriptions). Score each extracted character (e.g., antibiotic production, biofilm formation, host, methanogenesis products) using Relaxed precision, recall, and F1 against the gold standard. When a human baseline exists, compute the human matrix's per-character scores and compare the two systems with Student's t-Tests over per-character scores, requiring significance at p < 0.01. As an additional aggregate check, compare total relaxed hit scores (MicroPIE 445 vs. student 200 on the 46-description subset).

## Related tools

- **MicroPIE 0.1.0** (Text-mining extractor whose SVM-based character prediction and rule-based extraction outputs are being evaluated; run from the command line with -i (descriptions folder), -o (output folder), -m (models folder)) — github.com/biosemantics/micropie2
- **Maven** (Build system used to compile the MicroPIE project into a micropie-0.1.0.jar file before evaluation)
- **SVM (Support Vector Machine)** (Supervised classification component that identifies characters from sentences; its contribution to precision is measured by comparing performance with and without it) — github.com/biosemantics/micropie2

## Examples

```
java -cp micropieproject_lib -jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- MicroPIE must achieve higher Relaxed F1 than the student matrix on all 12 compared characters against the same gold standard
- Total relaxed hit score for MicroPIE (445) must exceed the student matrix (200) on the 46-description subset (a 123% difference)
- Student's t-Test over per-character Relaxed scores must show significance at p < 0.01
- Against the gold standard on the full set, MicroPIE should extract at least 21 of 42 characters (50%) with Relaxed F1 > 0.80 and achieve overall accuracy near 79.0%
- Ablation check: including the SVM character prediction component should improve overall performance, notably precision

## Limitations

- Nearly half of the designated characters had Relaxed_F1 scores not above 0.80 (16 characters fell between 0.50 and 0.80)
- Further development, including incorporation of ontologies, is needed to improve extraction for some character types; the authors state the system is not yet robust
- Evaluation depends on a hand-generated gold standard matrix, which is time-consuming to produce

## Evidence

- [results] MicroPIE outperformed students on all 12 compared characters; its total relaxed hit score (445) was 123% higher than the student matrix (200): [evidence span withheld — anchor status: not_found]
- [abstract] Relaxed F1 thresholds vs gold standard: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [results] Student's t-Test significance: [evidence span withheld — anchor status: not_found]
- [discussion] Gold standard evaluation framing: "Evaluation against a hand-generated gold standard matrix showed that MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters"
- [readme] Command line invocation: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
