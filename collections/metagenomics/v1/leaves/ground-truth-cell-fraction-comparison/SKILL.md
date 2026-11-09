---
name: ground-truth-cell-fraction-comparison
description: Use when you have SourceApp attribution outputs for spike-in mesocosms
  of known composition and need to score per-source sensitivity/specificity against
  Table S1 ground truth cellular fractions, e.g. to reproduce the finding that cow
  is the worst-performing source.
when_to_use_negative:
- Real impaired-waterway metagenomes with no spike-in ground truth — sensitivity/specificity
  cannot be computed without known mesocosm composition from Table S1
- Attribution CSVs are unavailable and the task is running the SourceApp pipeline
  itself (database curation, mapping) rather than evaluating its outputs
- Interpreting SourceApp portions as a mass balance — the paper states portions estimate
  proportions of cells based on genome equivalents, not a mass balance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3307
tools:
- name: SourceApp
  role: produces the attribution result tables (.csv) with per-source estimated cell
    fractions that are scored against the Table S1 ground truth
  repo: https://github.com/bglindner/SourceApp
- name: Python
  role: loads the attribution CSVs and ground-truth fractions and computes per-source
    sensitivity/specificity into source_attribution_accuracy.csv
- name: CoverM
  role: read-mapping engine (v0.7.0) underlying the genome coverage/abundance values
    that feed SourceApp's cell-fraction estimates being validated
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:446ed40b94f1658c9b39480e65be32dadb3db30de36656cd0eba0f789daabf36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ground-truth-cell-fraction-comparison

## Summary

Benchmarking skill that compares SourceApp's mesocosm source attributions against ground truth cellular fractions (Table S1), computing per-source sensitivity and specificity for the eight fecal source categories and exposing cow false negatives caused by an under-sampled cow genome database.

## When to use

Use when you have SourceApp attribution outputs for spike-in mesocosms of known composition and need to score per-source sensitivity/specificity against Table S1 ground truth cellular fractions, e.g. to reproduce the finding that cow is the worst-performing source.

## When NOT to use

- Real impaired-waterway metagenomes with no spike-in ground truth — sensitivity/specificity cannot be computed without known mesocosm composition from Table S1
- Attribution CSVs are unavailable and the task is running the SourceApp pipeline itself (database curation, mapping) rather than evaluating its outputs
- Interpreting SourceApp portions as a mass balance — the paper states portions estimate proportions of cells based on genome equivalents, not a mass balance

## Inputs

- SourceApp attribution result tables (.csv) for the n = 35 mesocosm metagenomes (PRJNA1161616)
- Ground truth cellular fraction values from Table S1 of the Supporting Information
- Mesocosm spike-in design metadata (which fecal source was added to each mesocosm and at what DAPI-counted cell concentration)

## Outputs

- source_attribution_accuracy.csv with one row per fecal source category (cat, chicken, cow, dog, goat, pig, sewage, septage) and per-source sensitivity/specificity
- Per-source false-negative tallies (e.g., cow false negatives across all spiked mesocosms)

## How to apply

Load the SourceApp attribution result tables (.csv) for all 35 mesocosm metagenomes together with the ground truth cellular fraction values from Table S1 of the Supporting Information, then reapply the paper's attribution logic: a source counts as 'present' only when at least one source-specific genome is detected AND its estimated cell fraction exceeds the limit of detection of 0.01% (optimal run parameters: 93% read-to-genome alignment identity, 70% alignment-to-read overlap, 0% target genome masking). For each of the eight fecal source categories (cat, chicken, cow, dog, goat, pig, sewage, septage), compute per-source sensitivity as the fraction of mesocosms in which the source was truly spiked in and correctly attributed, and per-source specificity from the false-positive attributions. Write one row per source to source_attribution_accuracy.csv. Validate that the table has 8 source rows with sensitivity values in [0, 1] and that pooled performance approaches the reported 0.902 sensitivity / 0.898 specificity. Confirm the paper's key pattern: cow is the worst performer with false negatives in all cow-spiked mesocosms, consistent with the cow database containing 43 species versus 759 for goat.

## Related tools

- **SourceApp** (produces the attribution result tables (.csv) with per-source estimated cell fractions that are scored against the Table S1 ground truth) — https://github.com/bglindner/SourceApp
- **Python** (loads the attribution CSVs and ground-truth fractions and computes per-source sensitivity/specificity into source_attribution_accuracy.csv)
- **CoverM** (read-mapping engine (v0.7.0) underlying the genome coverage/abundance values that feed SourceApp's cell-fraction estimates being validated) — https://github.com/wwood/CoverM

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('source_attribution_accuracy.csv'); assert len(t)==8 and t.sensitivity.between(0,1).all()"
```

## Evaluation signals

- source_attribution_accuracy.csv contains exactly 8 source rows (cat, chicken, cow, dog, goat, pig, sewage, septage) with sensitivity and specificity values in [0, 1]
- With optimal parameters (93% identity, 70% overlap, LOD 0.01%), pooled sensitivity ≈ 0.902 and specificity ≈ 0.898
- Every mesocosm containing cow spike-in signal is scored a false negative, matching the paper's under-sampling explanation (43 cow species vs 759 goat species)
- Sensitivity denominators count only mesocosms where the source was truly spiked in per Table S1 ground truth cellular fractions
- Sources scored 'present' require both a detected source-specific genome and an estimated cell fraction exceeding the 0.01% LOD

## Limitations

- Cow attributions are systematically underestimated because the cow fecal microbiome is under-sampled at the genomic level (43 species in the cow database vs 759 in the goat database)
- SourceApp portions do not represent a mass balance; even the best-performing parameters leave substantial residuals after linear correction (r2 = 0.64)
- Sensitivity/specificity near 0.90 overall masks large per-source variation, so pooled averages alone hide failures like the all-sample cow false negatives

## Evidence

- [abstract] SourceApp's performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes.: [evidence span withheld — anchor status: not_found]
- [results] 93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898"
- [results] false negatives for cows across all samples point to a poor understanding of the prokaryotic content of cow feces at the genomic level: "false negatives for cows across all samples point to a poor understanding of the prokaryotic content of cow feces at the genomic level"
- [results] substantial residuals remained after linear correction even with the best performing parameters: "substantial residuals remained after linear correction even with the best performing parameters"
- [methods] the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default): "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [readme] SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
