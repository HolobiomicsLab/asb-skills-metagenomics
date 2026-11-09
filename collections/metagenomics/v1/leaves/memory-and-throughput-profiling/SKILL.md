---
name: memory-and-throughput-profiling
description: Use when a tool's memory footprint (e.g. RawHash's large index for large
  genomes like the human genome) must be quantified and compared across seeding or
  preset modes, with peak RSS and indexing/mapping throughput tabulated side-by-side.
when_to_use_negative:
- When the goal is mapping accuracy evaluation rather than resource profiling — minimizer
  mode is slightly less accurate, so accuracy benchmarks need a different protocol
- When profiling small (e.g. yeast-sized) genomes where the index memory footprint
  is not a bottleneck and the minimizer trade-off is not meaningful
- When the tool being profiled does not expose alternative seeding/preset modes (e.g.
  plain Sigmap or UNCALLED without comparable presets)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Hash-based raw nanopore signal mapper whose index size and throughput are
    profiled; source of the minimizer-seeding `-x faster` preset
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Faster, more sensitive successor of RawHash whose default vs minimizer indexing
    modes are benchmarked for peak RSS and time
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:8572b7c65f504f72c5357635c138437609f22fb882f6bce39b5807278e385034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# memory-and-throughput-profiling

## Summary

Profiles peak memory (peak RSS) and wall-clock throughput of a bioinformatics indexing/mapping tool across configuration presets — here RawHash2's default seeding versus its minimizer-enabled `-x faster` preset — to quantify the time/memory trade-off of a technique such as minimizer seeding on large genomes.

## When to use

Use when a tool's memory footprint (e.g. RawHash's large index for large genomes like the human genome) must be quantified and compared across seeding or preset modes, with peak RSS and indexing/mapping throughput tabulated side-by-side.

## When NOT to use

- When the goal is mapping accuracy evaluation rather than resource profiling — minimizer mode is slightly less accurate, so accuracy benchmarks need a different protocol
- When profiling small (e.g. yeast-sized) genomes where the index memory footprint is not a bottleneck and the minimizer trade-off is not meaningful
- When the tool being profiled does not expose alternative seeding/preset modes (e.g. plain Sigmap or UNCALLED without comparable presets)

## Inputs

- Reference genome FASTA (e.g. human NA12878 D5 reference)
- RawHash2 binary built from a recursive clone of CMU-SAFARI/RawHash (bin/rawhash2)
- ONT k-mer pore model file (legacy r9.4 180mV 450bps 6-mer template_median68pA.model)
- Preset flag for minimizer mode (`-x faster`)

## Outputs

- Index file ref.ind from default-seeded indexing
- Second index file from minimizer-enabled (`-x faster`) indexing
- minimizer_benchmark.tsv tabulating peak memory (GB), throughput, relative speedup, and memory reduction per mode
- Peak RSS and wall-clock time measurements per run

## How to apply

Compile the tool from source (RawHash2 via `git clone --recursive https://github.com/CMU-SAFARI/RawHash.git` then `make`, producing bin/rawhash2). Build one index with default seeding — index the reference FASTA (e.g. human NA12878 D5) with the legacy r9.4 450bps 6-mer pore model (`extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model`) and 32 threads — recording peak RSS and wall-clock time. Repeat the identical indexing command with the minimizer-enabled `-x faster` preset into a second index file. Rationale: RawHash2's minimizer seeding mode is reported to be around 3X faster though slightly less accurate, and minimizer seeding is proposed as future work to mitigate the ~52GB index memory footprint for large genomes. Tabulate peak memory (GB), throughput, relative speedup, and memory reduction (e.g. into minimizer_benchmark.tsv); validate that the minimizer run is measurably faster, consistent with the ~3X speedup reported in the paper.

## Related tools

- **RawHash** (Hash-based raw nanopore signal mapper whose index size and throughput are profiled; source of the minimizer-seeding `-x faster` preset) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Faster, more sensitive successor of RawHash whose default vs minimizer indexing modes are benchmarked for peak RSS and time) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta
```

## Evaluation signals

- The minimizer-seeded (`-x faster`) indexing run completes measurably faster than the default-seeded run, consistent with the paper's reported ~3X speedup of the minimizer mode
- Both modes' peak memory and throughput values are present in minimizer_benchmark.tsv
- Peak RSS is recorded for both indexing runs (default and minimizer) so a relative memory reduction can be computed
- The identical command line, pore model, and thread count (e.g. 32 threads) are used for both runs so the comparison is controlled

## Limitations

- Minimizer seeding mode is slightly less accurate than the non-minimizer mode, so speed/memory gains come at an accuracy cost
- Minimizer seeding's effect on index memory footprint was proposed as future work and was not evaluated in the RawHash paper, so the expected memory reduction is not quantified in the publication
- Indexing parameters baked into the index cannot be changed afterwards, so separate index files must be built per preset

## Evidence

- [other] Minimizer seeding is implemented in RawHash but was not evaluated in the paper; it is proposed as future work that could mitigate the memory footprint of indexing large genomes.: [evidence span withheld — anchor status: not_found]
- [methods] RawHash2 minimizer seeding mode is around 3X faster but slightly less accurate than non-minimizer mode.: "which is slightly less accurate than the non-minimizer mode but much faster (around 3X)."
- [methods] Indexing example generates an index file ref.ind from ref.fasta with a k-mer model under extern and 32 threads.: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [intro] The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table.: "The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table."
