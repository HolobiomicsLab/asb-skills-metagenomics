---
name: metagenomic-abundance-extraction-from-classifier-outputs
description: Use when benchmarking taxonomic abundance estimators on admixture/food-sequencing
  samples where cross-reactivity (false-positive signal) between closely related species,
  e.g. cattle/water buffalo or sheep/goat, must be quantified per classifier.
when_to_use_negative:
- The goal is single-organism read classification (taxonomic labeling) rather than
  quantifying abundance false-positives across closely related species pairs
- Input samples contain no taxonomically close species pairs whose cross-reactivity
  can be measured (no cattle/water-buffalo- or sheep/goat-like comparisons)
- Only one classifier is available, so no relative FP-reduction factor can be computed
  against CLARK, Kraken2, or Kraken2+Bracken
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3168
tools:
- name: AFS-MetaCache
  role: Reference classifier whose abundance estimates (set via abundance_per) provide
    the low-FP baseline percentages (0.14% water buffalo, 0.05% goat on Klyo samples)
  repo: https://muellan.github.io/metacache/afs.html
- name: CLARK
  role: Competing classifier whose water-buffalo and goat FP percentages (4.2x and
    4.8x higher than AFS-MetaCache) are extracted for comparison
- name: Kraken2
  role: Competing classifier whose water-buffalo and goat FP percentages (5.6x and
    6.0x higher than AFS-MetaCache) are extracted for comparison
- name: Kraken2+Bracken
  role: Competing classifier whose water-buffalo and goat FP percentages (5.1x and
    10.0x higher than AFS-MetaCache) are extracted for comparison
- name: MetaCacheSpark
  role: Spark implementation of the minhashing classifier supporting the abundance_per
    query option used to produce abundance output on big data clusters
  repo: https://github.com/jmabuin/MetaCacheSpark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:0ece32ea4c7db1645cb875aa24c0742d34112adc09ddc34ceba5cc957241e600
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-abundance-extraction-from-classifier-outputs

## Summary

Extracts per-classifier abundance percentages for closely related species pairs (water buffalo vs cattle, goat vs sheep) from metagenomic classifier outputs and computes false-positive (FP) reduction factors to benchmark AFS-MetaCache against CLARK, Kraken2, and Kraken2+Bracken on all-food-sequencing (Klyo) samples.

## When to use

Use when benchmarking taxonomic abundance estimators on admixture/food-sequencing samples where cross-reactivity (false-positive signal) between closely related species, e.g. cattle/water buffalo or sheep/goat, must be quantified per classifier.

## When NOT to use

- The goal is single-organism read classification (taxonomic labeling) rather than quantifying abundance false-positives across closely related species pairs
- Input samples contain no taxonomically close species pairs whose cross-reactivity can be measured (no cattle/water-buffalo- or sheep/goat-like comparisons)
- Only one classifier is available, so no relative FP-reduction factor can be computed against CLARK, Kraken2, or Kraken2+Bracken

## Inputs

- Per-classifier abundance estimation output tables (AFS-MetaCache, CLARK, Kraken2, Kraken2+Bracken) from admixture quantification runs on the Klyo samples
- Per-tool query configurations, including AFS-MetaCache abundance levels set with the abundance_per option

## Outputs

- species_fp_comparison.csv with one row per classifier and sample, percentage fields for water buffalo vs cattle and goat vs sheep, and computed FP-reduction factors

## How to apply

Load the per-classifier quantification output tables produced by abundance estimation runs of AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken on the admixture (Klyo) samples. Parse each tool's abundance output to extract the detected percentage of water buffalo (cross-reacting with cattle) and goat (cross-reacting with sheep); for AFS-MetaCache use its abundance-estimation mode (the abundance_per option, e.g. at species level) as used in the article's query configurations. Compute the FP-reduction factor of AFS-MetaCache relative to each competitor by dividing that tool's reported FP percentage by AFS-MetaCache's; the Results text reports factors of approximately 4.2–5.1x for water buffalo and 4.8–10.0x for goat. Assemble one structured comparison row per classifier and sample, including percentage fields for both species pairs, and save as species_fp_comparison.csv. Rationale: AFS-MetaCache detects on average only 0.14% water buffalo and 0.05% goat on the Klyo samples, so factor values materially outside the reported ~4–10x range signal a parsing or extraction error.

## Related tools

- **AFS-MetaCache** (Reference classifier whose abundance estimates (set via abundance_per) provide the low-FP baseline percentages (0.14% water buffalo, 0.05% goat on Klyo samples)) — https://muellan.github.io/metacache/afs.html
- **CLARK** (Competing classifier whose water-buffalo and goat FP percentages (4.2x and 4.8x higher than AFS-MetaCache) are extracted for comparison)
- **Kraken2** (Competing classifier whose water-buffalo and goat FP percentages (5.6x and 6.0x higher than AFS-MetaCache) are extracted for comparison)
- **Kraken2+Bracken** (Competing classifier whose water-buffalo and goat FP percentages (5.1x and 10.0x higher than AFS-MetaCache) are extracted for comparison)
- **MetaCacheSpark** (Spark implementation of the minhashing classifier supporting the abundance_per query option used to produce abundance output on big data clusters) — https://github.com/jmabuin/MetaCacheSpark

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -a <abundance_level> -q threshold
```

## Evaluation signals

- species_fp_comparison.csv exists and contains percentage fields for both species pairs (water buffalo, goat) and all four classifiers
- All computed FP-reduction factors fall within the reported ~4–10x range (AFS-MetaCache factors: CLARK 4.2/4.8, Kraken2 5.6/6.0, Kraken2+Bracken 5.1/10.0)
- AFS-MetaCache baseline values match the article's reported averages of ~0.14% water buffalo and ~0.05% goat on the Klyo samples
- Parsed percentages are consistent with the abundance mode configuration (e.g. AFS-MetaCache abundance_per level) for each tool

## Limitations

- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken), hindering exact reproduction
- No availability statement exists for the raw sequencing data or reference sample datasets used for the admixture benchmarks
- No specific versions or identifiers of the reference genomes used for database construction are stated

## Evidence

- [results] On the Klyo samples, AFS-MetaCache detects only 0.14% water buffalo and 0.05% goat on average: [evidence span withheld — anchor status: not_found]
- [results] FP-reduction factors reported in Results: "higher by factors of 4.2 and 4.8, 5.6 and 6.0, and 5.1 and 10.0, respectively"
- [abstract] Superior false-positive rate claim: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [readme] abundance_per query option in MetaCacheSpark: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
- [discussion] Tool versions not documented: [evidence span withheld — anchor status: not_found]
