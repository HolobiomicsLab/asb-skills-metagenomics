---
name: food-ingredient-admixture-percentage-parsing
description: Use when quantifying ingredient percentages in food admixture samples
  and you need to compare false-positive cross-classification rates between closely
  related species across metagenomic classifiers.
when_to_use_negative:
- The sample contains only single-species (non-admixture) material, so no cross-reactivity
  FP signal can be measured
- The species pairs of interest are not closely related (cross-classification FP rates
  are not the analysis goal)
- Raw read classification accuracy (not ingredient percentage quantification) is the
  target output
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: AFS-MetaCache
  role: Primary classifier whose abundance estimation (abundance_per option) yields
    the low-FP ingredient percentages being extracted
  repo: https://muellan.github.io/metacache/afs.html
- name: CLARK
  role: Comparison classifier whose reported water buffalo/goat FP amounts are parsed
    to compute FP-reduction factors
- name: Kraken2
  role: Comparison classifier whose reported water buffalo/goat FP amounts are parsed
    to compute FP-reduction factors
- name: Kraken2+Bracken
  role: Comparison classifier (with abundance re-estimation) whose reported FP amounts
    are parsed to compute FP-reduction factors
- name: MetaCacheSpark
  role: Spark-based variant performing the same minhashing classification on big data
    clusters for the AFS pipeline
  repo: https://github.com/jmabuin/MetaCacheSpark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:b8d27d4704e17a38942207fe9cefbec9cddb63812e20535867811b29e2b7b236
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# food-ingredient-admixture-percentage-parsing

## Summary

Parse per-classifier abundance-estimation output to extract detected percentages of target ingredients in admixture samples, focusing on false-positive cross-reactivity between closely related species pairs (water buffalo vs. cattle, goat vs. sheep) from AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken benchmarks.

## When to use

Use when quantifying ingredient percentages in food admixture samples and you need to compare false-positive cross-classification rates between closely related species across metagenomic classifiers.

## When NOT to use

- The sample contains only single-species (non-admixture) material, so no cross-reactivity FP signal can be measured
- The species pairs of interest are not closely related (cross-classification FP rates are not the analysis goal)
- Raw read classification accuracy (not ingredient percentage quantification) is the target output

## Inputs

- Per-classifier abundance-estimation output tables (AFS-MetaCache, CLARK, Kraken2, Kraken2+Bracken) for admixture samples
- Admixture sample compositions (Klyo samples) with known ground-truth ingredient percentages
- Taxonomic level configuration for abundance estimation (e.g., AFS-MetaCache abundance_per option)

## Outputs

- species_fp_comparison.csv: structured comparison table with one row per classifier and sample, containing water buffalo and goat detected percentages and FP-reduction factors
- Per-classifier FP-reduction factors relative to AFS-MetaCache (expected in the ~4–10x range)

## How to apply

Load the per-classifier quantification tables produced by abundance estimation runs of AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken on the admixture (Klyo) samples. Parse each tool's abundance output to extract the detected percentage of water buffalo versus cattle and goat versus sheep per classifier and sample; for AFS-MetaCache the abundance levels are set with the abundance_per option. Compute the false-positive (FP) reduction factor of AFS-MetaCache relative to each competitor. The article reports AFS-MetaCache detects only 0.14% water buffalo and 0.05% goat on average, while CLARK, Kraken2, and Kraken2+Bracken report amounts higher by factors of 4.2/4.8, 5.6/6.0, and 5.1/10.0, respectively — i.e., roughly 4–10x lower FP rates. Assemble the percentages and computed factors into a structured comparison table with one row per classifier and sample.

## Related tools

- **AFS-MetaCache** (Primary classifier whose abundance estimation (abundance_per option) yields the low-FP ingredient percentages being extracted) — https://muellan.github.io/metacache/afs.html
- **CLARK** (Comparison classifier whose reported water buffalo/goat FP amounts are parsed to compute FP-reduction factors)
- **Kraken2** (Comparison classifier whose reported water buffalo/goat FP amounts are parsed to compute FP-reduction factors)
- **Kraken2+Bracken** (Comparison classifier (with abundance re-estimation) whose reported FP amounts are parsed to compute FP-reduction factors)
- **MetaCacheSpark** (Spark-based variant performing the same minhashing classification on big data clusters for the AFS pipeline) — https://github.com/jmabuin/MetaCacheSpark

## Examples

```
./metacache query afs_db klyo_samples/ -abundance_per species -out abundances.txt
```

## Evaluation signals

- species_fp_comparison.csv exists and contains percentage fields for both species pairs (water buffalo/cattle, goat/sheep) across all four classifiers
- All computed FP-reduction factors fall within the reported ~4–10x range
- AFS-MetaCache average FP values match the reported 0.14% water buffalo and 0.05% goat on the Klyo samples
- Each competitor's FP percentage equals the AFS-MetaCache value multiplied by its reported factor (e.g., Kraken2+Bracken goat ≈ 0.05% × 10.0)
- Table has exactly one row per classifier-and-sample combination with no missing percentage entries

## Limitations

- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), limiting reproducibility
- No availability statement for the raw sequencing data or reference sample datasets used for the admixture benchmarks
- FP factors were only demonstrated for closely related pairs (water buffalo/cattle, goat/sheep) on the Klyo samples; other pairs may behave differently

## Evidence

- [results] AFS-MetaCache FP rates for closely related species are roughly 4–10x lower: [evidence span withheld — anchor status: not_found]
- [abstract] Superiority in FP rate over competing classifiers: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [readme] Abundance estimation option in MetaCacheSpark query mode: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
- [discussion] Admixture benchmark reference samples: "we have applied our approach on a broad set of reference samples, containing admixtures of a set of food relevant ingredients (chicken, turkey, pork, beef, horse, sheep)"
