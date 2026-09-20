---
name: mesocosm-benchmark-evaluation
description: Use when validating a source attribution tool against mesocosms of known
  fecal spike-in composition, or when reproducing the per-source sensitivity table
  showing which fecal sources (e.g., cow) yield false negatives.
when_to_use_negative:
- When the metagenomes lack known ground truth (real impaired waterway samples) —
  sensitivity/specificity cannot be computed without spike-in compositions
- When the analysis goal is apportionment accuracy (cell fraction regression, r2 =
  0.64) rather than binary present/absent attribution
- When the source categories of interest rely on poorly represented databases (e.g.,
  cow, 43 species) — performance conclusions will be dominated by database sampling
  rather than tool behavior
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: Produces the per-metagenome source attribution result tables and estimated
    cell fractions that are scored against mesocosm ground truth
  repo: github.com/bglindner/SourceApp
- name: Python
  role: Implementation language for computing per-source sensitivity/specificity and
    writing source_attribution_accuracy.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:9ef987d0ecb380008885b82a10f2d6a044991084bd9664001b5ae207dc053d53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mesocosm-benchmark-evaluation

## Summary

Evaluates fecal source tracking performance (sensitivity/specificity per source category) by comparing tool attributions against known spike-in compositions of freshwater mesocosms, replicating the SourceApp benchmark that showed cow sources are systematically underestimated.

## When to use

Use when validating a source attribution tool against mesocosms of known fecal spike-in composition, or when reproducing the per-source sensitivity table showing which fecal sources (e.g., cow) yield false negatives.

## When NOT to use

- When the metagenomes lack known ground truth (real impaired waterway samples) — sensitivity/specificity cannot be computed without spike-in compositions
- When the analysis goal is apportionment accuracy (cell fraction regression, r2 = 0.64) rather than binary present/absent attribution
- When the source categories of interest rely on poorly represented databases (e.g., cow, 43 species) — performance conclusions will be dominated by database sampling rather than tool behavior

## Inputs

- SourceApp attribution result tables (.csv) for all mesocosm metagenomes
- Ground truth cellular fraction values from Table S1 of the Supporting Information
- Mesocosm spike-in composition metadata (n = 35 shotgun-sequenced mesocosms from Lake Lanier water)
- SourceApp parameters used (93% identity, 70% query coverage, 0% masking, LOD 0.01%)

## Outputs

- source_attribution_accuracy.csv with one row per fecal source (8 rows) containing sensitivity and specificity values in [0, 1]
- Per-source false-negative rate summary identifying cow as worst performer

## How to apply

Load the SourceApp attribution result tables (.csv) for all mesocosm metagenomes together with the ground truth cellular fraction values from Table S1 of the Supporting Information. Apply the paper's attribution logic: score a source as 'present' when at least one source-specific genome is detected (sequence breadth above the -l/--limit-threshold at 93% identity, 70% query coverage) and its estimated cell fraction exceeds the limit of detection of 0.01%. For each fecal source category (cat, chicken, cow, dog, goat, pig, sewage, septage), compute per-source sensitivity as the fraction of mesocosms where the source was truly spiked in and correctly attributed, and specificity from false-positive attributions. Write one row per source to source_attribution_accuracy.csv and validate the reported pattern — cow shows the worst per-source performance because all cow-spiked mesocosms were false negatives, reflecting under-sampling of the cow fecal microbiome (759 goat species vs 43 cow species in the databases).

## Related tools

- **SourceApp** (Produces the per-metagenome source attribution result tables and estimated cell fractions that are scored against mesocosm ground truth) — github.com/bglindner/SourceApp
- **Python** (Implementation language for computing per-source sensitivity/specificity and writing source_attribution_accuracy.csv)

## Examples

```
present = df['genome_detected'].any() and df['cell_fraction'] > 0.0001  # LOD 0.01%
```

## Evaluation signals

- The output table contains exactly 8 source rows (cat, chicken, cow, dog, goat, pig, sewage, septage)
- All sensitivity and specificity values fall in [0, 1]
- Cow exhibits the highest false-negative rate — all mesocosms containing cow spike-in signal were reported as false negatives
- Optimal-parameter benchmark reproduces specificity 0.898 and sensitivity 0.902 (near 0.90 overall)
- Attribution calls match the LOD rule: a source is 'present' only when a source-specific genome is detected AND its cell fraction exceeds 0.01%

## Limitations

- Cow false negatives reflect under-sampling of the cow fecal microbiome in the reference database (goat: 759 species vs cow: 43 species), so per-source metrics confound tool performance with database coverage
- SourceApp portions do not represent a mass balance but estimate proportions of cells based on genome equivalents
- Substantial residuals remained after linear correction of cell fraction estimates (r2 = 0.64), so attribution metrics do not capture apportionment error
- Persistence of species-level marker genomes in the environment is largely unknown, which can affect environmental (non-mesocosm) applicability

## Evidence

- [abstract] SourceApp's performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes.: "SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes."
- [methods] Limit of detection rule: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [results] Optimal parameters and benchmark result: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898"
- [results] Cow false negatives explanation: "false negatives for cows across all samples point to a poor understanding of the prokaryotic content of cow feces at the genomic level"
- [results] Portions are not mass balance: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories"
