---
name: real-time-mapping-behavior-analysis
description: Use when comparing how early competing real-time raw-signal mappers stop
  sequencing a read (bases sequenced per read) or how many signal chunks they consume
  per read, to estimate adaptive-sampling time/cost benefits.
when_to_use_negative:
- Input reads are already basecalled FASTQ/FASTA — this skill measures signal-prefix
  and chunk consumption from raw signal mapping, not sequence-level alignment statistics
- Data is r10.4.1 — UNCALLED (one of the comparison baselines) only supports legacy
  r9.4.1 data, so the time/cost comparison cannot be run
- The goal is mapping accuracy or throughput rather than sequencing time/cost proxies
  — use the accuracy/throughput evaluation workflow instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: RawHash
  role: Primary mapper whose real-time stopping behavior (read prefix length, chunks
    per read) is being measured; stops as soon as a read prefix maps
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline for the sequenced-bases-per-read comparison; its PAF template is
    shared by the evaluation
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline for the chunks-per-read comparison (RawHash uses on average 1.58x
    more chunks per read); shares the PAF output template
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:2422115acb41e8c813072304003191f9d25886d05bed7c691647b118b43613a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# real-time-mapping-behavior-analysis

## Summary

Quantifies the real-time sequencing behavior of raw nanopore signal mappers (RawHash, UNCALLED, Sigmap) by measuring the sequenced read prefix length and number of signal chunks consumed per mapped read, which proxy sequencing time and cost under adaptive sampling. This reproduces the paper's Table 4 comparison showing RawHash stops earlier than UNCALLED on large genomes but consumes ~1.58x more chunks than Sigmap.

## When to use

Use when comparing how early competing real-time raw-signal mappers stop sequencing a read (bases sequenced per read) or how many signal chunks they consume per read, to estimate adaptive-sampling time/cost benefits.

## When NOT to use

- Input reads are already basecalled FASTQ/FASTA — this skill measures signal-prefix and chunk consumption from raw signal mapping, not sequence-level alignment statistics
- Data is r10.4.1 — UNCALLED (one of the comparison baselines) only supports legacy r9.4.1 data, so the time/cost comparison cannot be run
- The goal is mapping accuracy or throughput rather than sequencing time/cost proxies — use the accuracy/throughput evaluation workflow instead

## Inputs

- FAST5 raw nanopore signal datasets (test/data/d1_sars-cov-2_r94 through d5_human_na12878_r94)
- PAF mapping outputs from RawHash, UNCALLED, and Sigmap in the modified UNCALLED/Sigmap PAF template
- Reference genome FASTA indices for each mapper (RawHash .ind, UNCALLED BWA index, Sigmap .si/.pt)

## Outputs

- Per-dataset, per-tool average sequenced read prefix length (bases per mapped read) for RawHash vs UNCALLED
- Per-dataset, per-tool average number of sequenced signal chunks per mapped read for RawHash vs Sigmap
- Combined comparison table written to table4_prefix_chunk_stats.csv (mirroring Table 4)

## How to apply

Run RawHash, UNCALLED, and Sigmap over the same FAST5 datasets (e.g. D1 SARS-CoV-2 through D5 human NA12878, r9.4.1) so each emits PAF records in the shared UNCALLED/Sigmap PAF template. Parse each mapped-read record with the pafstats tooling to extract the sequenced read prefix length in bases — RawHash stops as soon as a read prefix maps to the reference genome. For the RawHash vs UNCALLED comparison, average the sequenced prefix length over all mapped reads per dataset and tool, which estimates sequencing time/cost since shorter prefixes mean the basecaller ejects reads sooner. For the RawHash vs Sigmap comparison, instead count the number of sequenced signal chunks consumed per mapped read (Sigmap processes in chunks rather than prefixes) and average per dataset and tool. Assemble the per-dataset, per-tool averages into a single table mirroring Table 4 and write it to table4_prefix_chunk_stats.csv.

## Related tools

- **RawHash** (Primary mapper whose real-time stopping behavior (read prefix length, chunks per read) is being measured; stops as soon as a read prefix maps) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline for the sequenced-bases-per-read comparison; its PAF template is shared by the evaluation) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline for the chunks-per-read comparison (RawHash uses on average 1.58x more chunks per read); shares the PAF output template) — https://github.com/haowenz/sigmap

## Examples

```
uncalled map -t 16 E.coli fast5_list.txt > uncalled_out.paf
```

## Evaluation signals

- RawHash consumes a larger number of sequenced chunks per read than Sigmap (paper reports ~1.58x more on average)
- RawHash sequences a shorter read prefix than UNCALLED on large genomes (e.g. Green Algae and Human datasets)
- Computed per-dataset, per-tool averages fall in plausible ranges for each dataset (D1-D5, SARS-CoV-2 through human NA12878)
- All three mappers emit PAF records parseable by the shared pafstats tooling in the UNCALLED/Sigmap template
- Output table4_prefix_chunk_stats.csv has one row per dataset × tool × metric with no missing mapped-read aggregates

## Limitations

- Chunk-based time/cost comparisons are proportional: RawHash's 1.58x higher chunk usage can proportionally worsen its time and cost relative to Sigmap
- UNCALLED only applies to legacy r9.4.1 data, restricting the comparison to r9.4.1 datasets
- The prefix-length metric (vs UNCALLED) and chunk-count metric (vs Sigmap) are not directly comparable to each other; each is tied to how the respective baseline consumes signal

## Evidence

- [results] RawHash reduces sequencing time and cost for large genomes (e.g. Green Algae and Human) by using shorter read prefixes than UNCALLED: [evidence span withheld — anchor status: not_found]
- [results] RawHash uses on average 1.58x more chunks per read than Sigmap, which can proportionally worsen its time and cost relative to Sigmap: [evidence span withheld — anchor status: not_found]
- [methods] We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information"
- [readme] Note that UNCALLED can only be applied to legacy r9.4.1 data. For r10.4.1 data try [ReadFish]: [evidence span withheld — anchor status: external_doc]
- [readme] RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
