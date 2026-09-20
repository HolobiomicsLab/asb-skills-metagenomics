---
name: pafstats-runtime-profiling
description: Use when comparing raw nanopore signal mappers (RawHash, UNCALLED, Sigmap)
  on D1–D5 FAST5 datasets and you need per-tool CPU/real-time and bases-per-second
  throughput ratios to test real-time feasibility.
when_to_use_negative:
- Datasets are R10.4.1 rather than legacy r9.4.1 data — UNCALLED (and thus its pafstats
  comparison baseline) can only be applied to legacy r9.4.1 data
- The goal is mapping accuracy evaluation rather than runtime/throughput profiling
  — pafstats extracts timing metrics, not alignment accuracy
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Primary mapper under test; produces mapping.paf outputs whose throughput is
    profiled with pafstats
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Comparison mapper and source of the pafstats utility used to extract CPU time,
    real time, and throughput from PAF outputs
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Comparison streaming raw-signal mapper run on the same datasets and references
    to produce PAF mappings for throughput comparison
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:ceb0182a84699e7ed03f170f822446bded58f25324e2dc89afbb723ab082044e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pafstats-runtime-profiling

## Summary

Profile raw-signal mapping throughput by running UNCALLED's pafstats utility over PAF outputs from RawHash, UNCALLED, and Sigmap to extract CPU time, real time, and bases-per-second, enabling head-to-head benchmarking against nanopore sequencing rates.

## When to use

Use when comparing raw nanopore signal mappers (RawHash, UNCALLED, Sigmap) on D1–D5 FAST5 datasets and you need per-tool CPU/real-time and bases-per-second throughput ratios to test real-time feasibility.

## When NOT to use

- Datasets are R10.4.1 rather than legacy r9.4.1 data — UNCALLED (and thus its pafstats comparison baseline) can only be applied to legacy r9.4.1 data
- The goal is mapping accuracy evaluation rather than runtime/throughput profiling — pafstats extracts timing metrics, not alignment accuracy

## Inputs

- PAF mapping outputs from RawHash/RawHash2, UNCALLED, and Sigmap on D1–D5 FAST5 datasets
- FAST5 raw nanopore signal files for five target genomes (including SARS-CoV-2 and human NA12878)
- Reference genome FASTA files with prebuilt indexes (ref.ind for RawHash, .si/.pt for Sigmap)
- Legacy R9.4 6-mer pore model (template_median68pA.model)

## Outputs

- Per-tool, per-dataset CPU time, real time, and bases-processed-per-second throughput from pafstats
- Aggregated throughput table across D1–D5
- RawHash-to-UNCALLED and RawHash-to-Sigmap average throughput ratios (expected ~25.8× and ~3.4×)

## How to apply

Map the same FAST5 datasets with all three tools under identical conditions (32 threads, matching references) so PAF outputs are directly comparable. Build each reference index first: RawHash2 with the legacy R9.4 6-mer pore model (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) using genome-size presets (-x viral for SARS-CoV-2, -x sensitive for small genomes, -x fast for the >500M human NA12878 set); Sigmap indexing likewise uses an ONT r9.4 6-mer pore model. Run UNCALLED's pafstats utility on each PAF output to extract CPU time, real time, and bases-processed-per-second per tool per dataset. Aggregate the metrics into a table and compute RawHash-to-UNCALLED and RawHash-to-Sigmap average throughput ratios across D1–D5, then check against the reported 25.8× and 3.4× values.

## Related tools

- **RawHash** (Primary mapper under test; produces mapping.paf outputs whose throughput is profiled with pafstats) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Comparison mapper and source of the pafstats utility used to extract CPU time, real time, and throughput from PAF outputs) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Comparison streaming raw-signal mapper run on the same datasets and references to produce PAF mappings for throughput comparison) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -x fast -t 32 -i ref.ind -r reads.fast5 > mapping.paf && uncalled pafstats mapping.paf
```

## Evaluation signals

- Measured RawHash-to-UNCALLED average throughput ratio across D1–D5 matches the reported 25.8×
- Measured RawHash-to-Sigmap average throughput ratio matches the reported 3.4×
- Sigmap's human-genome (D5) throughput is below 1× the nanopore sequencing rate (reported as 0.7× for human genome mapping)
- All three tools ran with identical conditions: same datasets, same references, 32 threads, and PAF outputs consumable by pafstats
- pafstats yields nonzero bases-processed-per-second and real/CPU time for every tool-dataset pair

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data; R10.4.1 datasets require other tools (e.g., ReadFish)
- Sigmap cannot achieve real-time throughput for large genomes: only 0.7× and 0.6× nanopore throughput for human genome mapping and relative abundance estimation
- Throughput depends on genome size and preset choice; results from small-genome sets (e.g., viral) do not transfer to the >500M human NA12878 set

## Evidence

- [abstract] 25.8x vs UNCALLED and 3.4x vs Sigmap throughput ratios: "RawHash provides (i) 25.8× and 3.4× better average throughput and (ii) significantly better accuracy for large genomes, respectively."
- [methods] pafstats extracts CPU time, real time, throughput from PAF: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [intro] RawHash matches nanopore sequencing throughput for large genomes: "such that it can match the throughput of nanopore sequencing even when analyzing large genomes (e.g., human genome."
- [readme] UNCALLED restricted to legacy r9.4.1 data: [evidence span withheld — anchor status: external_doc]
- [readme] Sigmap indexing uses ONT pore model from extern folder: [evidence span withheld — anchor status: external_doc]
