---
name: species-quantification-comparison-water-buffalo-goat
description: Use when quantifying cross-hybridization false positives between closely
  related food-animal genomes (e.g. water buffalo vs cattle, goat vs sheep) across
  multiple metagenomic abundance classifiers on admixture samples.
when_to_use_negative:
- The task requires de novo abundance estimation from raw reads rather than comparison
  of already-produced classifier quantification outputs
- The species of interest are not closely related pairs where cross-reactivity (e.g.
  cattle/water buffalo, sheep/goat) drives the false-positive signal
- The benchmark involves taxonomic read labeling rather than quantitative abundance
  comparison across multiple classifiers
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier whose abundance estimates (enabled via the abundance_per
    option) serve as the low-FP reference in the comparison
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark-based variant used to run build and query with abundance estimation
    (-a abundance_per) on a big data cluster
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Established metagenomic classifier benchmarked as a comparator for water buffalo
    and goat false-positive rates
- name: Kraken2
  role: Established k-mer classifier benchmarked as a comparator for water buffalo
    and goat false-positive rates
- name: Kraken2+Bracken
  role: Kraken2 with Bracken abundance re-estimation benchmarked as a comparator for
    water buffalo and goat false-positive rates
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:7e017ec3c5ad372c2667780e74ddaf73b954fec2ee15847048c782205fc9dc75
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# species-quantification-comparison-water-buffalo-goat

## Summary

Extracts and compares false-positive abundance estimates for closely related species pairs — water buffalo cross-reacting with cattle and goat cross-reacting with sheep — as reported by AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken in the All-Food-Sequencing sausage quantification benchmarks (Klyo samples).

## When to use

Use when quantifying cross-hybridization false positives between closely related food-animal genomes (e.g. water buffalo vs cattle, goat vs sheep) across multiple metagenomic abundance classifiers on admixture samples.

## When NOT to use

- The task requires de novo abundance estimation from raw reads rather than comparison of already-produced classifier quantification outputs
- The species of interest are not closely related pairs where cross-reactivity (e.g. cattle/water buffalo, sheep/goat) drives the false-positive signal
- The benchmark involves taxonomic read labeling rather than quantitative abundance comparison across multiple classifiers

## Inputs

- Abundance estimation output tables from AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken on the admixture (Klyo sausage) samples
- Per-sample FASTQ/FASTA read files from the sausage admixture benchmarks
- Reference genome database(s) built from food-relevant eukaryotic and microbial genomes

## Outputs

- species_fp_comparison.csv with one row per classifier and sample, containing water buffalo %, goat %, and computed FP-reduction factors

## How to apply

Load the per-classifier abundance estimation output tables from AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken runs on the admixture (Klyo sausage) samples. For AFS-MetaCache, abundance estimation is enabled at a selectable taxonomic level via the abundance_per option (e.g. -a species); for MetaCacheSpark, query mode can be tuned with -q precise/threshold/fast/very_fast and candidate filtering via -g hits_greater_than. Parse each tool's output to extract the detected percentage of water buffalo (against a cattle background) and goat (against a sheep background) per sample, then compute the false-positive reduction factor of AFS-MetaCache relative to each competitor. The paper reports AFS-MetaCache detects only ~0.14% water buffalo and ~0.05% goat on average, with CLARK, Kraken2, and Kraken2+Bracken FP amounts higher by factors of 4.2/4.8, 5.6/6.0, and 5.1/10.0 respectively. Assemble one row per classifier and sample into a structured comparison table (species_fp_comparison.csv).

## Related tools

- **AFS-MetaCache** (C++ workstation classifier whose abundance estimates (enabled via the abundance_per option) serve as the low-FP reference in the comparison) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark-based variant used to run build and query with abundance estimation (-a abundance_per) on a big data cluster) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Established metagenomic classifier benchmarked as a comparator for water buffalo and goat false-positive rates)
- **Kraken2** (Established k-mer classifier benchmarked as a comparator for water buffalo and goat false-positive rates)
- **Kraken2+Bracken** (Kraken2 with Bracken abundance re-estimation benchmarked as a comparator for water buffalo and goat false-positive rates)

## Examples

```
./metacache query refseq myReads1.fa myReads2.fa -pairfiles -abundance_per species -out results.txt
```

## Evaluation signals

- Output table species_fp_comparison.csv exists and contains percentage fields for both species pairs (water buffalo vs cattle, goat vs sheep) across all four classifiers
- All computed FP-reduction factors of AFS-MetaCache versus the competing classifiers fall within the reported ~4–10x range (4.2/4.8, 5.6/6.0, 5.1/10.0)
- AFS-MetaCache average FP values are approximately 0.14% for water buffalo and 0.05% for goat on the Klyo samples
- One row per classifier and sample; no missing percentage entries for either species pair

## Limitations

- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), limiting reproducibility
- No availability statement for the raw sequencing data or reference sample datasets used for the admixture benchmarks
- No specific versions or identifiers of the reference genomes used for database construction are stated

## Evidence

- [results] AFS-MetaCache detects only 0.14% water buffalo and 0.05% goat on average: [evidence span withheld — anchor status: not_found]
- [results] higher by factors of 4.2 and 4.8, 5.6 and 6.0, and 5.1 and 10.0: [evidence span withheld — anchor status: not_found]
- [abstract] superior in terms of false-positive rate and quantification accuracy: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [readme] -a,--abundance_per <arg>: "-a,--abundance_per <arg>             [QUERY] Indicates if use the
                                          abundance estimation feature and at
                                          which level."
- [intro] estimate the abundance of organisms at a selectable taxonomic level: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
