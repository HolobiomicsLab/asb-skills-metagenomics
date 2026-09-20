---
name: cross-tool-performance-ratio-analysis
description: Use when comparing mapping throughput/speed of raw-signal nanopore mappers
  across tools on the same FAST5 datasets and reference genomes, e.g., verifying whether
  each tool exceeds nanopore sequencing throughput on the human genome.
when_to_use_negative:
- The task is accuracy/precision evaluation of mappings rather than speed or throughput
  comparison — this skill measures CPU/real time and bases-per-second only.
- Only one mapping tool is being evaluated with no second tool to form a comparative
  ratio.
- Input data is not legacy R9.4.1 raw signal data, since UNCALLED can only be applied
  to legacy r9.4.1 data.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Primary mapper being benchmarked; produces PAF mappings of raw signals for
    throughput measurement
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline mapper run under identical 32-thread conditions; its pafstats utility
    extracts throughput metrics from PAF outputs
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Second baseline mapper run on the same datasets and references to form the
    RawHash-to-Sigmap throughput ratio
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:9b6cdf7fc1b3ea4b0e3174c35f29ef068b143252f3f28b7173eaa09474c5bc14
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
yes_rate: 0.875
n_claims: 16
---

# cross-tool-performance-ratio-analysis

## Summary

Benchmark multiple raw nanopore signal mappers (RawHash, UNCALLED, Sigmap) under identical conditions (same FAST5 datasets, references, thread counts), extract per-tool CPU/real time and throughput with UNCALLED's pafstats, and compute average throughput ratios against the published baselines (25.8x vs UNCALLED, 3.4x vs Sigmap).

## When to use

Use when comparing mapping throughput/speed of raw-signal nanopore mappers across tools on the same FAST5 datasets and reference genomes, e.g., verifying whether each tool exceeds nanopore sequencing throughput on the human genome.

## When NOT to use

- The task is accuracy/precision evaluation of mappings rather than speed or throughput comparison — this skill measures CPU/real time and bases-per-second only.
- Only one mapping tool is being evaluated with no second tool to form a comparative ratio.
- Input data is not legacy R9.4.1 raw signal data, since UNCALLED can only be applied to legacy r9.4.1 data.

## Inputs

- D1-D5 raw nanopore FAST5 datasets (including SARS-CoV-2 and human NA12878 >500M sets)
- Reference genome FASTA files for each of the five target genomes
- RawHash2 index files (ref.ind) built with the legacy R9.4 6-mer pore model
- PAF mapping outputs from RawHash, UNCALLED, and Sigmap

## Outputs

- Per-tool, per-dataset pafstats metrics (CPU time, real time, bases-processed-per-second throughput)
- Aggregated throughput comparison table across D1-D5
- RawHash-to-UNCALLED and RawHash-to-Sigmap average throughput ratios
- Validation verdicts against published ratios (25.8x vs UNCALLED, 3.4x vs Sigmap)

## How to apply

Build a reference index (ref.ind) for each target genome with rawhash2 using the legacy R9.4 6-mer pore model (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) and 32 threads. Map each D1-D5 FAST5 dataset to its matching reference with rawhash2 at 32 threads using genome-size presets from the README (-x viral for the SARS-CoV-2 set, -x sensitive for small-genome sets, -x fast for the >500M human NA12878 set), writing mapping.paf outputs. Run UNCALLED and Sigmap on the same datasets and references with 32 threads so all three tools produce PAF mappings under identical conditions. Run UNCALLED's pafstats utility on each PAF output to extract CPU time, real time, and bases-processed-per-second throughput per tool per dataset, then aggregate the metrics into a table and compute RawHash-to-UNCALLED and RawHash-to-Sigmap average throughput ratios across D1-D5.

## Related tools

- **RawHash** (Primary mapper being benchmarked; produces PAF mappings of raw signals for throughput measurement) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline mapper run under identical 32-thread conditions; its pafstats utility extracts throughput metrics from PAF outputs) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Second baseline mapper run on the same datasets and references to form the RawHash-to-Sigmap throughput ratio) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -x fast -t 32 -i ref.ind -f NA12878.fast5 -o mapping.paf && uncalled pafstats mapping.paf
```

## Evaluation signals

- Measured RawHash-to-UNCALLED average throughput ratio across D1-D5 matches the reported 25.8x.
- Measured RawHash-to-Sigmap average throughput ratio across D1-D5 matches the reported 3.4x.
- Sigmap's human-genome (D5) throughput falls below 1x the nanopore sequencing rate (reported as ~0.7x for human genome mapping).
- All three tools were run with the same 32 threads on the same FAST5 datasets and matching references, producing PAF outputs under identical conditions.
- pafstats outputs contain CPU time, real time, and bases-processed-per-second for every tool-dataset pair.

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data; r10.4.1 data requires other tools such as ReadFish or ONT's builtin adaptive sampling.
- Sigmap cannot perform real-time analysis for large genomes, achieving only ~0.7x and ~0.6x the throughput of a nanopore for human genome mapping and relative abundance estimation.
- Throughput ratios are averages across D1-D5; per-dataset ratios may deviate from the published averages.

## Evidence

- [abstract] Published average throughput ratios are the validation targets: "When compared to the state-of-the-art techniques, UNCALLED and Sigmap, RawHash provides (i) 25.8× and 3.4× better average throughput"
- [results] Sigmap fails real-time mapping on the human genome: [evidence span withheld — anchor status: not_found]
- [methods] Indexing command with pore model and threads: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [readme] UNCALLED is restricted to legacy R9.4.1 data: [evidence span withheld — anchor status: external_doc]
- [readme] RawHash goal is to match nanopore throughput on large genomes: "efficiently and accurately maps the raw signals to the reference genome such that it can match the throughput of nanopore sequencing even when analyzing large genomes"
