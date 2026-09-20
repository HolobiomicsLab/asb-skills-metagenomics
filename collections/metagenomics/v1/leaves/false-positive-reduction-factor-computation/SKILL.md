---
name: false-positive-reduction-factor-computation
description: Use when comparing false-positive rates of metagenomic classifiers on
  admixed food samples containing closely related species pairs (e.g. buffalo/cattle,
  goat/sheep), where quantification outputs must be cross-parsed and converted to
  FP-reduction factors.
when_to_use_negative:
- The samples contain no closely related species pairs, so no cross-reactivity false
  positives (e.g. water buffalo/cattle, goat/sheep) can be measured.
- The goal is quantification accuracy or sensitivity benchmarking rather than false-positive
  rate comparison — the article treats FP rate as a separate axis from quantification
  accuracy.
- Only a single classifier's output is available; the FP-reduction factor is a cross-classifier
  ratio and requires all four tools' outputs.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: AFS-MetaCache
  role: Baseline classifier whose abundance-estimation output (water buffalo, goat
    percentages) is compared against competitors; its low FP rates define the reduction
    factors
  repo: https://muellan.github.io/metacache/afs.html
- name: CLARK
  role: Competitor classifier whose reported water buffalo/goat percentages provide
    the comparison points (factors 4.2 and 4.8)
- name: Kraken2
  role: Competitor classifier providing cross-reactivity FP percentages (factors 5.6
    and 6.0)
- name: Kraken2+Bracken
  role: Competitor abundance-corrected pipeline providing cross-reactivity FP percentages
    (factors 5.1 and 10.0)
- name: MetaCacheSpark
  role: Spark-based implementation of the same minhashing method; its FP rates are
    compared alongside AFS-MetaCache
  repo: https://github.com/jmabuin/MetaCacheSpark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:81fbb3e6f3cc02ec6577d3302a582ddc2dfd5cfe3f504d63e9883b9ec86b3971
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# false-positive-reduction-factor-computation

## Summary

Extracts false-positive cross-reactivity percentages (water buffalo detected in cattle-only content, goat in sheep-only content) from the abundance-estimation outputs of AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken on the Klyo sausage quantification samples, and computes AFS-MetaCache's FP-reduction factors relative to each competing classifier.

## When to use

Use when comparing false-positive rates of metagenomic classifiers on admixed food samples containing closely related species pairs (e.g. buffalo/cattle, goat/sheep), where quantification outputs must be cross-parsed and converted to FP-reduction factors.

## When NOT to use

- The samples contain no closely related species pairs, so no cross-reactivity false positives (e.g. water buffalo/cattle, goat/sheep) can be measured.
- The goal is quantification accuracy or sensitivity benchmarking rather than false-positive rate comparison — the article treats FP rate as a separate axis from quantification accuracy.
- Only a single classifier's output is available; the FP-reduction factor is a cross-classifier ratio and requires all four tools' outputs.

## Inputs

- AFS-MetaCache abundance-estimation output (per-taxon percentages) from the Klyo sausage samples, obtained with the abundance_per option
- CLARK quantification output tables for the same admixture samples
- Kraken2 quantification output tables for the same admixture samples
- Kraken2+Bracken abundance-corrected output tables for the same admixture samples

## Outputs

- species_fp_comparison.csv — one row per classifier and sample, containing detected percentages for water buffalo vs. cattle and goat vs. sheep plus the computed FP-reduction factors of AFS-MetaCache relative to each competitor

## How to apply

Run the abundance-estimation mode of all four classifiers on the admixture (Klyo) samples, with AFS-MetaCache using the abundance_per option to enable quantification at a selectable taxonomic level. Parse each classifier's per-taxon abundance output to extract the detected percentage of water buffalo (cross-reacting with cattle) and goat (cross-reacting with sheep) for every sample. For each classifier and species pair, compute the FP-reduction factor as (competitor FP percentage) / (AFS-MetaCache FP percentage), since the article's Results text reports these as factors of approximately 4-10x. Assemble percentages and computed factors into a structured table (species_fp_comparison.csv) with one row per classifier and sample. The rationale is that AFS-MetaCache's filtering of target references based on sequence coverage suppresses cross-reactivity between closely related genomes, whereas CLARK, Kraken2, and Kraken2+Bracken report substantially higher amounts of the non-target species.

## Related tools

- **AFS-MetaCache** (Baseline classifier whose abundance-estimation output (water buffalo, goat percentages) is compared against competitors; its low FP rates define the reduction factors) — https://muellan.github.io/metacache/afs.html
- **CLARK** (Competitor classifier whose reported water buffalo/goat percentages provide the comparison points (factors 4.2 and 4.8))
- **Kraken2** (Competitor classifier providing cross-reactivity FP percentages (factors 5.6 and 6.0))
- **Kraken2+Bracken** (Competitor abundance-corrected pipeline providing cross-reactivity FP percentages (factors 5.1 and 10.0))
- **MetaCacheSpark** (Spark-based implementation of the same minhashing method; its FP rates are compared alongside AFS-MetaCache) — https://github.com/jmabuin/MetaCacheSpark

## Examples

```
./metacache query afsdb klyo_reads.fq -out results.txt -abundance_per species
```

## Evaluation signals

- All computed FP-reduction factors fall within the reported ~4-10x range.
- AFS-MetaCache water buffalo FP averages ~0.14% and goat FP averages ~0.05% on the Klyo samples.
- Per-competitor factors match the reported values: CLARK 4.2/4.8, Kraken2 5.6/6.0, Kraken2+Bracken 5.1/10.0.
- species_fp_comparison.csv exists and contains percentage fields for both species pairs and all four classifiers.
- Each competitor's FP percentage is strictly higher than AFS-MetaCache's for both species pairs, giving factors > 1.

## Limitations

- The article provides no availability statement for the raw sequencing data or reference sample datasets used for the admixture benchmarks, complicating reproduction.
- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), limiting exact reimplementation of the comparison.
- FP-reduction factors are only reported for the closely related species pairs measured (water buffalo/cattle, goat/sheep); generalization to other species pairs is untested.

## Evidence

- [abstract] superiority over competitors: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [intro] abundance estimation and coverage filtering augmentations: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [discussion] FP-rate comparison in discussion: "The comparison to the established metagenomics tools Kraken2, CLARK, and Kraken2+Bracken shows that AFS-MetaCache and MetaCacheSpark are superior in terms of false positive (FP) rates."
- [readme] abundance_per query option: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
- [methods] MetaCacheSpark query execution: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
