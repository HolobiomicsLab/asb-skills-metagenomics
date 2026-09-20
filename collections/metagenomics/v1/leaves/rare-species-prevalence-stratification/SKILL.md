---
name: rare-species-prevalence-stratification
description: Use when per-sample shotgun sequencing is shallow and you need to verify
  that genome cover plateaus (saturates) for rare or low-prevalence taxa once coverage
  is composited across like samples from the same environment.
when_to_use_negative:
- Alignments were not generated against the Web of Life database (calculate_coverages.py
  currently only works with WoL)
- You already have a feature/abundance table and only want to filter it — the README
  warns this approach decreases apparent read counts rather than redistributing them
  as Woltka would
- Per-sample coverage is already deep enough that saturation is evident without compositing
  across like samples
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: calculate_coverages.py
  role: Aggregates unique genome coverage of each WoL target genome across all like
    samples from .sam alignments
  repo: https://github.com/biocore/zebra_filter
- name: bowtie2
  role: Generates the .sam alignments of shotgun reads against WoL reference genomes
- name: SHOGUN
  role: Alternative aligner for producing .sam alignments (bowtie2 alignment option)
- name: Woltka
  role: Optional integration for downstream abundance-table generation from the alignments
  repo: https://github.com/qiyunzhu/woltka
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00758-22
    title: 'Zebra: Static and Dynamic Genome Cover Thresholds with Overlapping References'
  source_content_hash: sha256:335c5bfd3760c25c1b7abcbe53e34925b5f5378de4fa44a76aa412afb9ca6b85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/zebra_filter
  tools:
  - Python
  - SHOGUN
  - Woltka
  - bowtie2
  - calculate_coverages.py
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Rare-species prevalence stratification of composited genome cover

## Summary

Aggregates unique genome coverage across like samples (e.g., human fecal metagenomes aligned to the Web of Life reference genomes) and stratifies genomes by prevalence/rarity to confirm that composited genome cover saturates even for rare or low-prevalence species.

## When to use

Use when per-sample shotgun sequencing is shallow and you need to verify that genome cover plateaus (saturates) for rare or low-prevalence taxa once coverage is composited across like samples from the same environment.

## When NOT to use

- Alignments were not generated against the Web of Life database (calculate_coverages.py currently only works with WoL)
- You already have a feature/abundance table and only want to filter it — the README warns this approach decreases apparent read counts rather than redistributing them as Woltka would
- Per-sample coverage is already deep enough that saturation is evident without compositing across like samples

## Inputs

- Directory of .sam alignment files from bowtie2 or SHOGUN (bowtie2 alignment option) against Web of Life (WoL) reference genomes
- WoL genome metadata file (databases/WoL/metadata.tsv)

## Outputs

- Coverage .tsv with columns gotu, covered_length, total_length, coverage_ratio, strain (e.g., genome_cover_saturation.tsv)

## How to apply

Generate .sam alignments of shotgun reads against Web of Life (WoL) reference genomes using bowtie2 or SHOGUN (bowtie2 alignment option), then run calculate_coverages.py on the directory of .sam files with the WoL metadata (-d databases/WoL/metadata.tsv). The script aggregates the unique genome coverage of each target genome across all like samples — only unique coverage counts, since repeated reads over the same genomic location do not add coverage — producing 'artificially deep' shotgun sequencing per genome. From the output table compute coverage_ratio = covered_length / total_length for each genome (gotu). Stratify genomes by prevalence/rarity in the cohort and inspect composited coverage_ratio values to confirm cover plateaus even for rare species, indicating the composite behaves like deep shotgun sequencing. Write per-genome results as a .tsv with columns gotu, covered_length, total_length, coverage_ratio, strain.

## Related tools

- **calculate_coverages.py** (Aggregates unique genome coverage of each WoL target genome across all like samples from .sam alignments) — https://github.com/biocore/zebra_filter
- **bowtie2** (Generates the .sam alignments of shotgun reads against WoL reference genomes)
- **SHOGUN** (Alternative aligner for producing .sam alignments (bowtie2 alignment option))
- **Woltka** (Optional integration for downstream abundance-table generation from the alignments) — https://github.com/qiyunzhu/woltka

## Examples

```
python calculate_coverages.py -i test_data/ -o test_data/sample_output.tsv -d databases/WoL/metadata.tsv
```

## Evaluation signals

- Output .tsv exists and contains all required columns: gotu, covered_length, total_length, coverage_ratio, strain
- Every row's coverage_ratio lies in [0, 1] and equals covered_length / total_length
- Rare-species genomes show saturated (plateaued) coverage_ratio values comparable to prevalent ones after compositing across like samples
- Coverage is unique-coverage only: re-running on alignments where repeated reads hit the same loci yields unchanged covered_length

## Limitations

- calculate_coverages.py currently only works with the Web of Life database
- Only unique genome coverage is counted — multiple reads covering the same genome location do not increase coverage
- The compositing assumption requires that like samples from the same environment share similar microbes; it is invalid across heterogeneous environments

## Evidence

- [abstract] Saturation for rare species via like-sample compositing: "By compositing the metric across like samples, genome cover saturates even for rare species."
- [intro] Unique coverage counting rationale: "only unique coverage is counted, multiple reads covering the same location of the genome will not increase the coverage"
- [readme] Tool invocation and inputs: "Calculates the aggregated unique coverage of each target genome from all the input alignments together."
- [readme] Output schema: "gotu	covered_length	total_length	coverage_ratio	strain"
- [readme] coverage_ratio definition: "coverage_ratio: Portion of hit genome covered. Calculated by covered_length / total_length"
