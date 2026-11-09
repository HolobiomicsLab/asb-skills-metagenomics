---
name: computational-performance-comparison
description: Use when comparing memory footprint and throughput of hash-based raw
  signal mappers on large genomes (e.g., human NA12878/D5), or when validating claimed
  speedups such as RawHash2 minimizer seeding (~3X).
when_to_use_negative:
- Input is sequence-based reads rather than raw nanopore signals — RawHash's hash-based
  comparisons target raw signal mapping, not FASTQ mappers
- The question is about mapping accuracy rather than computational cost — minimizer
  mode is explicitly slightly less accurate, so accuracy-focused questions need a
  different evaluation
- Genome is small (e.g., yeast) where the ~52GB index memory footprint for large genomes
  is not the bottleneck
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RawHash
  role: Primary hash-based raw nanopore signal mapper whose throughput (25.8x vs UNCALLED,
    3.4x vs Sigmap) and memory footprint are being benchmarked
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Faster, more sensitive successor build used to compare minimizer (-x faster)
    vs default seeding during indexing
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: State-of-the-art baseline raw signal mapper for throughput comparison on legacy
    r9.4 data
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming raw signal mapper for throughput comparison
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:ca717527f42b5e42a4d4137886ed74368272688720b76af5b1ff35b419348dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# computational-performance-comparison

## Summary

Benchmarking protocol for quantifying peak memory (peak RSS) and wall-clock throughput of raw nanopore signal mapping tools (RawHash/RawHash2, UNCALLED, Sigmap) on large genomes, including preset-level comparisons such as minimizer versus default seeding.

## When to use

Use when comparing memory footprint and throughput of hash-based raw signal mappers on large genomes (e.g., human NA12878/D5), or when validating claimed speedups such as RawHash2 minimizer seeding (~3X).

## When NOT to use

- Input is sequence-based reads rather than raw nanopore signals — RawHash's hash-based comparisons target raw signal mapping, not FASTQ mappers
- The question is about mapping accuracy rather than computational cost — minimizer mode is explicitly slightly less accurate, so accuracy-focused questions need a different evaluation
- Genome is small (e.g., yeast) where the ~52GB index memory footprint for large genomes is not the bottleneck

## Inputs

- Reference genome FASTA (e.g., human NA12878 D5 reference)
- RawHash2 binary built via `make` from a recursive clone (bin/rawhash2)
- ONT pore model file (legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- Raw nanopore signal reads in FAST5, POD5, SLOW5, or BLOW5 format (for mapping throughput runs)

## Outputs

- Index files from each seeding mode (e.g., ref.ind per preset)
- Benchmark table (minimizer_benchmark.tsv) with peak memory (GB), indexing/mapping throughput, speedup, and memory reduction per mode
- Peak RSS and wall-clock timing measurements per run

## How to apply

Compile RawHash2 from a recursive clone of the CMU-SAFARI repository with `make` to obtain bin/rawhash2. Index the reference FASTA (e.g., human NA12878/D5) with the legacy r9.4 450bps 6-mer pore model (`extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model`) and 32 threads, profiling peak RSS and wall-clock time; then repeat the identical indexing command with the minimizer-enabled `-x faster` preset. Tabulate peak memory (GB) and throughput for both modes, computing relative speedup and memory reduction into a TSV. As cross-tool context, the paper reports RawHash achieving 25.8x and 3.4x better average throughput than UNCALLED and Sigmap, so comparisons should be run on identical FAST5/POD5/SLOW5/BLOW5 inputs and reference genomes. Validation relies on reproducing reported relative ratios (e.g., minimizer mode ~3X faster) rather than absolute numbers.

## Related tools

- **RawHash** (Primary hash-based raw nanopore signal mapper whose throughput (25.8x vs UNCALLED, 3.4x vs Sigmap) and memory footprint are being benchmarked) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Faster, more sensitive successor build used to compare minimizer (-x faster) vs default seeding during indexing) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (State-of-the-art baseline raw signal mapper for throughput comparison on legacy r9.4 data) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming raw signal mapper for throughput comparison) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta
```

## Evaluation signals

- Both default and minimizer indexing runs complete and yield peak RSS and wall-clock values recorded in minimizer_benchmark.tsv
- Minimizer (-x faster) indexing is measurably faster than default seeding, consistent with the reported ~3X speedup
- Throughput ratios against baselines fall in the expected regime (RawHash 25.8x vs UNCALLED, 3.4x vs Sigmap on large genomes)
- Indexing commands are identical between modes except for the seeding preset, so memory/throughput deltas are attributable to seeding alone
- Accuracy is checked separately since minimizer mode is slightly less accurate than non-minimizer mode

## Limitations

- Minimizer seeding in RawHash2 is faster (~3X) but slightly less accurate than the non-minimizer mode
- RawHash's index for large genomes such as human has a large (~52GB) memory footprint; minimizer mitigation was proposed as future work and was not evaluated in the paper
- UNCALLED can only be applied to legacy r9.4.1 data, limiting cross-tool comparisons to that chemistry

## Evidence

- [abstract] RawHash provides (i) 25.8× and 3.4× better average throughput and (ii) significantly better accuracy for large genomes, respectively.: "When compared to the state-of-the-art techniques, UNCALLED and Sigmap, RawHash provides (i) 25.8× and 3.4× better average throughput"
- [methods] Minimizer seeding mode in RawHash2 is much faster (around 3X) but slightly less accurate.: "which is slightly less accurate than the non-minimizer mode but much faster (around 3X)."
- [methods] RawHash2 is a more sensitive and faster raw signal mapping mechanism with substantial improvements over RawHash.: "We have released RawHash2, a more sensitive and faster raw signal mapping mechanism with substantial improvements over RawHash."
- [methods] Indexing command example generating ref.ind with a k-mer model and 32 threads.: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [readme] UNCALLED only supports legacy r9.4.1 data.: [evidence span withheld — anchor status: external_doc]
