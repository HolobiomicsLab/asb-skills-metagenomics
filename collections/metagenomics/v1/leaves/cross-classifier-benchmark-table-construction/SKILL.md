---
name: cross-classifier-benchmark-table-construction
description: Use when multiple metagenomic classifiers have been run on admixture/quantification
  samples containing cross-reacting species pairs, and you need to extract per-tool
  false-positive percentages and compute relative FP-reduction factors.
when_to_use_negative:
- The benchmark has only one classifier (no competitor output to compute FP-reduction
  factors against).
- The samples contain no closely related cross-reacting species pairs, so cross-reactivity
  FP rates are not the quantity of interest.
- You need quantification accuracy (true-abundance error) rather than false-positive
  cross-reactivity, which requires a different comparison table.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: AFS-MetaCache
  role: Reference classifier whose FP percentages are extracted; its abundance_per
    query option controls the taxonomic level of abundance estimation
  repo: https://muellan.github.io/metacache/afs.html
- name: CLARK
  role: Competitor classifier whose water buffalo/goat FP amounts are extracted as
    baselines (higher by a factor of 4.2 and 4.8)
- name: Kraken2
  role: Competitor classifier whose FP amounts are extracted as baselines (higher
    by factors of 5.6 and 6.0)
- name: Kraken2+Bracken
  role: Competitor classifier with abundance re-estimation whose FP amounts are extracted
    as baselines (higher by factors of 5.1 and 10.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:1716f25ce46ced51671d03b4ac46e66da405957b16a1e746ed9b77826d0d5537
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-classifier-benchmark-table-construction

## Summary

Builds a structured false-positive comparison table quantifying cross-reactivity of closely related species pairs (water buffalo vs. cattle, goat vs. sheep) across AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken in all-food-sequencing admixture benchmarks, and computes the FP-reduction factor of AFS-MetaCache relative to each competitor (~4–10x on the Klyo samples).

## When to use

Use when multiple metagenomic classifiers have been run on admixture/quantification samples containing cross-reacting species pairs, and you need to extract per-tool false-positive percentages and compute relative FP-reduction factors.

## When NOT to use

- The benchmark has only one classifier (no competitor output to compute FP-reduction factors against).
- The samples contain no closely related cross-reacting species pairs, so cross-reactivity FP rates are not the quantity of interest.
- You need quantification accuracy (true-abundance error) rather than false-positive cross-reactivity, which requires a different comparison table.

## Inputs

- Per-classifier abundance estimation output tables (AFS-MetaCache, CLARK, Kraken2, Kraken2+Bracken) from admixture samples such as the Klyo samples
- AFS-MetaCache query configuration with abundance levels set via the abundance_per option
- Species-pair definition (water buffalo vs. cattle; goat vs. sheep)

## Outputs

- species_fp_comparison.csv with one row per classifier and sample, percentage fields for both species pairs across all four classifiers, and computed FP-reduction factors

## How to apply

Load the per-classifier quantification tables produced by the abundance estimation runs of AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken on the admixture samples. Parse each tool's abundance output to extract the detected percentage of water buffalo (cross-reacting with cattle) and goat (cross-reacting with sheep) per sample; for AFS-MetaCache runs the abundance level is set via the abundance_per option in the query configuration. Compute the FP-reduction factor of AFS-MetaCache against each competitor by dividing its reported percentages by the competing tool's values; the Results text reports these as factors of approximately 4–10x (e.g. AFS-MetaCache detects only 0.14% water buffalo and 0.05% goat on average on the Klyo samples, while Kraken2+Bracken reports values higher by factors of 5.1 and 10.0). Assemble one row per classifier and sample with both species-pair percentage fields and the computed factors, then save as species_fp_comparison.csv.

## Related tools

- **AFS-MetaCache** (Reference classifier whose FP percentages are extracted; its abundance_per query option controls the taxonomic level of abundance estimation) — https://muellan.github.io/metacache/afs.html
- **CLARK** (Competitor classifier whose water buffalo/goat FP amounts are extracted as baselines (higher by a factor of 4.2 and 4.8))
- **Kraken2** (Competitor classifier whose FP amounts are extracted as baselines (higher by factors of 5.6 and 6.0))
- **Kraken2+Bracken** (Competitor classifier with abundance re-estimation whose FP amounts are extracted as baselines (higher by factors of 5.1 and 10.0))

## Examples

```
spark-submit --class com.github.metacachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -q threshold -a species -n 4 -p 16 -b 10000 sample.fastq
```

## Evaluation signals

- species_fp_comparison.csv exists and contains percentage fields for both species pairs (water buffalo vs. cattle, goat vs. sheep) and all four classifiers
- All computed FP-reduction factors fall within the reported ~4–10x range
- AFS-MetaCache averages on the Klyo samples reproduce the reference values of 0.14% water buffalo and 0.05% goat
- Every classifier/sample row has non-missing values for both species-pair percentages

## Limitations

- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), limiting reproducibility of the FP comparison.
- No availability statement exists for the raw sequencing data or reference sample datasets used for the admixture benchmarks, so the underlying per-sample percentages may not be re-derivable.
- FP amounts are specific to the closely related pairs water buffalo/cattle and goat/sheep; they do not generalize to other species pairs or samples.

## Evidence

- [abstract] Abstract superiority claim: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [discussion] Discussion FP-rate comparison: "The comparison to the established metagenomics tools Kraken2, CLARK, and Kraken2+Bracken shows that AFS-MetaCache and MetaCacheSpark are superior in terms of false positive (FP) rates."
- [readme] MetaCacheSpark abundance option: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
