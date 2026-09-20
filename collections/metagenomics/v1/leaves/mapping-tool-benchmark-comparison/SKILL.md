---
name: mapping-tool-benchmark-comparison
description: 'Use when comparing raw nanopore signal mappers for adaptive-sampling
  benefits: PAF mappings from multiple tools (e.g., RawHash, UNCALLED, Sigmap) are
  available on the same FAST5 datasets and you need per-tool averages of read prefix
  length or chunks consumed per mapped read.'
when_to_use_negative:
- The tool comparison involves read accuracy or throughput rather than sequencing
  time/cost proxies (read prefix length / chunks per read) — use the accuracy/throughput
  evaluation workflow instead.
- Input data is r10.4.1; UNCALLED can only be applied to legacy r9.4.1 data, so this
  three-tool comparison is invalid for newer chemistries.
- Only one mapper's PAF output is available — the prefix-length and chunk-consumption
  comparisons require matched mappings from RawHash, UNCALLED, and Sigmap on the same
  datasets.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Hash-based raw nanopore signal mapper being benchmarked; reports mappings
    in a PAF template with read prefix/chunk info
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline raw signal mapper; its PAF output template is parsed to compute average
    sequenced read prefix length
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming signal mapper; its PAF output is parsed to compute average
    sequenced chunks per read
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:c0fb7da724cd21d325dff5836f9580a1d3050192145a972b17ce912f6ff1a410
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# mapping-tool-benchmark-comparison

## Summary

A benchmarking skill for comparing real-time nanopore raw-signal mapping tools (RawHash, UNCALLED, Sigmap) on sequencing time/cost proxies, measured as average sequenced read prefix length (bases) and average number of sequenced signal chunks per read from modified PAF outputs.

## When to use

Use when comparing raw nanopore signal mappers for adaptive-sampling benefits: PAF mappings from multiple tools (e.g., RawHash, UNCALLED, Sigmap) are available on the same FAST5 datasets and you need per-tool averages of read prefix length or chunks consumed per mapped read.

## When NOT to use

- The tool comparison involves read accuracy or throughput rather than sequencing time/cost proxies (read prefix length / chunks per read) — use the accuracy/throughput evaluation workflow instead.
- Input data is r10.4.1; UNCALLED can only be applied to legacy r9.4.1 data, so this three-tool comparison is invalid for newer chemistries.
- Only one mapper's PAF output is available — the prefix-length and chunk-consumption comparisons require matched mappings from RawHash, UNCALLED, and Sigmap on the same datasets.

## Inputs

- PAF mapping outputs from RawHash, UNCALLED, and Sigmap (modified PAF template shared by UNCALLED and Sigmap)
- FAST5 read directories test/data/d1_sars-cov-2_r94 through test/data/d5_human_na12878_r94 (D1-D5 datasets, r9.4.1 chemistry)

## Outputs

- table4_prefix_chunk_stats.csv: per-dataset, per-tool averages of sequenced bases per read (vs UNCALLED) and sequenced chunks per read (vs Sigmap), mirroring the paper's Table 4

## How to apply

Load the PAF mapping outputs produced by RawHash, UNCALLED, and Sigmap on the D1-D5 datasets (e.g., test/data/d1_sars-cov-2_r94 through d5_human_na12878_r94 FAST5 directories), which follow the modified PAF template shared by UNCALLED and Sigmap. Parse each mapped-read record with pafstats tooling to extract the sequenced read prefix length in bases at the time mapping stopped, since RawHash stops as soon as a read prefix maps to the reference genome. For the RawHash vs UNCALLED comparison, average the sequenced prefix length over all mapped reads per dataset and tool; for the RawHash vs Sigmap comparison, count the number of sequenced signal chunks consumed per mapped read and average them. Assemble the per-dataset, per-tool averages into a single table mirroring the paper's Table 4 (e.g., table4_prefix_chunk_stats.csv). Validate that RawHash consumes a larger number of sequenced chunks than Sigmap (reported as ~1.58x more chunks per read) while sequencing a shorter read prefix than UNCALLED on large genomes such as Green Algae and Human.

## Related tools

- **RawHash** (Hash-based raw nanopore signal mapper being benchmarked; reports mappings in a PAF template with read prefix/chunk info) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline raw signal mapper; its PAF output template is parsed to compute average sequenced read prefix length) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming signal mapper; its PAF output is parsed to compute average sequenced chunks per read) — https://github.com/haowenz/sigmap

## Examples

```
uncalled map -t 16 E.coli fast5_list.txt > uncalled_out.paf
```

## Evaluation signals

- RawHash averages a larger number of sequenced chunks per read than Sigmap (reported ~1.58x more) across the D1-D5 datasets.
- RawHash averages a shorter sequenced read prefix (bases) than UNCALLED on large genomes (Green Algae, Human).
- All three tools' PAF records parse cleanly under the same modified PAF template with pafstats tooling.
- Computed per-dataset averages fall in plausible ranges for each dataset and reproduce the direction of the paper's Table 4.

## Limitations

- RawHash's use of more chunks per read than Sigmap can proportionally worsen its sequencing time and cost relative to Sigmap.
- UNCALLED only supports legacy r9.4.1 data, restricting the comparison to r9.4.1 FAST5 datasets.
- In UNCALLED real-time mode, a MinKNOW API bug may prevent proper read ejection; chunk-size changes require restarting UNCALLED before sequencing begins.

## Evidence

- [results] RawHash reduces sequencing time and cost for large genomes by using shorter read prefixes than UNCALLED, but uses on average 1.58x more chunks per read than Sigmap: [evidence span withheld — anchor status: not_found]
- [readme] modified PAF template from README: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [readme] uncalled map PAF output example: [evidence span withheld — anchor status: external_doc]
- [readme] r9.4.1 chemistry limitation: [evidence span withheld — anchor status: external_doc]
- [abstract] RawHash real-time throughput for large genomes: "Our evaluations show that RawHash is the only tool that can provide high accuracy and high throughput for analyzing large genomes in real-time."
