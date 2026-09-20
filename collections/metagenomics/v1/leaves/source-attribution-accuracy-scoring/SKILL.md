---
name: source-attribution-accuracy-scoring
description: Use when you have source tracking predictions (per-sample source calls
  and cell-fraction apportionments) plus known spike-in compositions and need to quantify
  attribution sensitivity/specificity.
when_to_use_negative:
- Predictions come from tools that lack calibrated presence/absence calls (e.g., decOM,
  metaSourceTracker reported specificity of 0), so 0.01%-LOD presence rules do not
  transfer
- No ground-truth spike-in composition exists — use environmental validation (e.g.,
  E. coli Colilert concentrations) instead of sensitivity/specificity scoring
- Input is already an aggregate accuracy table rather than per-sample attribution
  calls
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SourceApp
  role: Produces the per-sample source attribution, apportionment, and cell-fraction
    tables that are scored against ground truth
  repo: https://github.com/bglindner/SourceApp
- name: Python
  role: 'Implements the scoring logic: presence calling at the 0.01% LOD, per-source
    and overall sensitivity/specificity computation, CSV output'
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:154d86c1ba217431d82b7b6e171cb864e8261a1b8df27a3a3207e581b13f7342
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - FEAST
  - Kraken2
  - MicrobeCensus
  - Python
  - SourceApp
  - bwa
  - decOM
  - fastp
  - kmtricks
  - metaSourceTracker
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# source-attribution-accuracy-scoring

## Summary

Scores metagenomic fecal source tracking predictions (SourceApp attribution/apportionment tables) against ground-truth spike-in mesocosm compositions to compute overall and per-source sensitivity and specificity, plus apportionment correlation.

## When to use

Use when you have source tracking predictions (per-sample source calls and cell-fraction apportionments) plus known spike-in compositions and need to quantify attribution sensitivity/specificity.

## When NOT to use

- Predictions come from tools that lack calibrated presence/absence calls (e.g., decOM, metaSourceTracker reported specificity of 0), so 0.01%-LOD presence rules do not transfer
- No ground-truth spike-in composition exists — use environmental validation (e.g., E. coli Colilert concentrations) instead of sensitivity/specificity scoring
- Input is already an aggregate accuracy table rather than per-sample attribution calls

## Inputs

- SourceApp per-sample source attribution tables (CSV, one row per mesocosm/source)
- SourceApp apportionment and estimated cell-fraction tables (CSV, genome-equivalent fractions via --use-geq)
- Ground-truth mesocosm compositions (Supporting Information Table S1, cellular fractions from DAPI counting at construction)
- List of fecal source categories: cat, chicken, cow, dog, goat, pig, sewage, septage

## Outputs

- Per-source accuracy/sensitivity table (source_attribution_accuracy.csv, 8 source rows)
- Overall sensitivity and specificity metrics (sourceapp_benchmark_metrics.csv)
- Comparative sensitivity/specificity/apportionment-correlation table across tools (tool_comparison_table.csv)

## How to apply

Load SourceApp's per-sample attribution and fraction CSVs and the ground-truth cellular fractions (Supporting Information Table S1 for the PRJNA1161616 mesocosms). Apply the paper's presence rule: a source is scored 'present' only when at least one source-specific genome is detected AND its estimated cell fraction exceeds the limit of detection of 0.01% (min-frac 0.0001, with --use-geq genome-equivalent fractions). For each fecal source category (cat, chicken, cow, dog, goat, pig, sewage, septage) compute per-source sensitivity as the fraction of mesocosms where the source was truly spiked in and correctly attributed, and per-source specificity from false-positive calls. Then compute overall sensitivity/specificity across all 35 mesocosms and compare against the tuned-parameter benchmark (0.93 identity, 0.7 query coverage, 0% masking, 0.01% LOD → 0.902 sensitivity, 0.898 specificity). For apportionment accuracy, correlate estimated cell fractions with ground truth (SourceApp r = 0.86 vs FEAST 0.79, decOM 0.74, metaSourceTracker 0.60).

## Related tools

- **SourceApp** (Produces the per-sample source attribution, apportionment, and cell-fraction tables that are scored against ground truth) — https://github.com/bglindner/SourceApp
- **Python** (Implements the scoring logic: presence calling at the 0.01% LOD, per-source and overall sensitivity/specificity computation, CSV output)

## Evaluation signals

- Overall sensitivity and specificity should be near 0.90 when scored with tuned parameters (reported: 0.902 sensitivity, 0.898 specificity)
- Per-source table must contain exactly 8 source rows with sensitivity values in [0, 1]
- Cow should show the worst per-source performance (all cow-containing mesocosms scored false negatives), reflecting the 43-species cow database vs 759-species goat database
- Apportionment correlation with ground-truth cell fractions should exceed alternative tools (SourceApp 0.86 vs FEAST 0.79, decOM 0.74, metaSourceTracker 0.60)
- Alternative tools should reproduce the reported pattern of high sensitivity but very poor specificity (0.30 for FEAST, 0 for decOM and metaSourceTracker)

## Limitations

- Cow sources are systematically underestimated because the cow fecal microbiome is under-sampled in the reference database (43 species vs 759 for goat)
- SourceApp cannot infer contamination age or identify nonconservative (growth-capable) populations, and species-level marker genomes have largely unknown environmental persistence
- Portions are genome-equivalent cell-fraction estimates, not a mass balance; linear correction of cell-fraction estimates still leaves substantial residuals (r2 = 0.64)

## Evidence

- [abstract] Parameter tuning revealed sensitivity and specificity near 0.90 overall, which exceeded all alternative tools.: "Parameter tuning revealed sensitivity and specificity near 0.90 overall, which exceeded all alternative tools."
- [results] 93% identity / 70% overlap / 0.01% LOD yielded 0.898 specificity and 0.902 sensitivity.: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898"
- [methods] Source must exceed the limit of detection to be called present.: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [abstract] Cow underestimation due to under-sampled cow fecal microbiome.: "SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes."
- [readme] SourceApp reports prokaryotic cell fraction metrics with --use-geq.: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called)"
