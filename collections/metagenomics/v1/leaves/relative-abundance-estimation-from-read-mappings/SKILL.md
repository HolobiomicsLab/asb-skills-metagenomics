---
name: relative-abundance-estimation-from-read-mappings
description: Use when a mock community is pooled from FAST5 read sets and raw signals
  must be mapped to a combined multi-genome reference to obtain per-genome relative
  abundance estimates and compare their accuracy to ground truth.
when_to_use_negative:
- Input is basecalled FASTQ/FASTA reads rather than raw nanopore signals — use a base-level
  mapper (e.g., minimap2) instead, since RawHash, UNCALLED, and Sigmap map raw electrical
  signal
- Data is not legacy r9.4.1 chemistry — UNCALLED explicitly supports only legacy r9.4.1
  data, and the 6-mer pore models used here are R9.4-specific
- The goal is reference-free comparison or de novo assembly — use Rawsamble for all-vs-all
  raw-signal overlap finding instead of reference-based abundance estimation
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: RawHash/RawHash2
  role: 'Primary raw-signal mapper: indexes the combined reference (ref.ind) with
    the R9.4 6-mer pore model, maps pooled FAST5 signals to produce the relative abundance
    estimates, and provides real-time Sequence Until stopping'
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline raw-signal mapper run under default settings on the same pooled reads
    and combined reference to provide the comparison abundance estimates
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: 'Baseline streaming raw-signal mapper: indexes the reference with its ONT
    pore model and maps the same pooled reads for the accuracy comparison'
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:e34ed05da9f8a20b9f297044229958e86ebc8ad52e8101fd711816c60c4f3113
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash/RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# relative-abundance-estimation-from-read-mappings

## Summary

Estimate per-genome relative abundances of a mock community by mapping raw nanopore signals (FAST5) against a concatenated multi-species reference with raw-signal mappers (RawHash, UNCALLED, Sigmap), tallying mapped reads per reference genome, and computing Euclidean distance of the abundance vector to the ground-truth mixing proportions.

## When to use

Use when a mock community is pooled from FAST5 read sets and raw signals must be mapped to a combined multi-genome reference to obtain per-genome relative abundance estimates and compare their accuracy to ground truth.

## When NOT to use

- Input is basecalled FASTQ/FASTA reads rather than raw nanopore signals — use a base-level mapper (e.g., minimap2) instead, since RawHash, UNCALLED, and Sigmap map raw electrical signal
- Data is not legacy r9.4.1 chemistry — UNCALLED explicitly supports only legacy r9.4.1 data, and the 6-mer pore models used here are R9.4-specific
- The goal is reference-free comparison or de novo assembly — use Rawsamble for all-vs-all raw-signal overlap finding instead of reference-based abundance estimation

## Inputs

- Pooled FAST5 raw nanopore read sets from the mock community (e.g., SARS-CoV-2, yeast, green algae, human NA12878, R9.4 chemistry)
- Combined reference FASTA with concatenated genomes annotated by species/taxonomy ID
- R9.4 450bps 6-mer pore model file (e.g., legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- Ground-truth mixing proportions of the mock community

## Outputs

- Raw-signal mapping output in UNCALLED-style modified PAF format (mapping.paf)
- Per-genome relative abundance estimates per tool, with values in [0,1]
- Euclidean distance per tool between estimated and ground-truth abundance vectors (table3_relative_abundance.csv)
- Optional: estimation distances and read-usage fractions for Sequence Until evaluation (Tables 5 and 6 CSVs)

## How to apply

Pool the FAST5 read sets (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94) into the mock community and concatenate their reference genomes, annotated with species/taxonomy IDs, into a single combined reference FASTA. Build the raw-signal index of the combined reference — for RawHash/RawHash2 with the legacy R9.4 450bps 6-mer pore model (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) and 32 threads, producing ref.ind; Sigmap similarly indexes with its ONT pore model submodule. Map the pooled raw signals with rawhash2 using 32 threads and the size-appropriate preset (-x sensitive for small-genome mixes, -x fast for large genomes), emitting UNCALLED-style modified-PAF alignments; run UNCALLED and Sigmap under their default raw-signal mapping settings on the same inputs. Tally mapped reads per reference genome from each tool's mapping output, convert per-genome counts into relative abundance estimates in [0,1], and compute the Euclidean distance between each tool's per-genome abundance vector and the ground-truth mixing proportions. RawHash should yield the smallest Euclidean distance; the same pipeline extends to Sequence Until evaluation where randomly shuffled subsamples (0.01%–25%) emulate truncated sequencing runs and RawHash's real-time stopping mechanism records the fraction of each read's signal consumed before stopping (~7% of the full read set).

## Related tools

- **RawHash/RawHash2** (Primary raw-signal mapper: indexes the combined reference (ref.ind) with the R9.4 6-mer pore model, maps pooled FAST5 signals to produce the relative abundance estimates, and provides real-time Sequence Until stopping) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline raw-signal mapper run under default settings on the same pooled reads and combined reference to provide the comparison abundance estimates) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming raw-signal mapper: indexes the reference with its ONT pore model and maps the same pooled reads for the accuracy comparison) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -x sensitive -t 32 ref.ind pooled_reads.fast5 > mapping.paf
```

## Evaluation signals

- Output CSV exists and contains one row per genome in the combined reference plus one Euclidean-distance row per tool
- All abundance values fall within [0,1]
- RawHash's Euclidean distance to ground truth is not worse than those of UNCALLED and Sigmap, consistent with the paper's accuracy comparison
- For the Sequence Until variant, the fraction of reads consumed before stopping approximates the reported ~7% of the entire read set
- Estimation distances for RawHash vs UNCALLED match the relative ordering reported in Tables 5 and 6 (e.g., RawHash with 0.1% of reads more accurate than UNCALLED with the entire read set)

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data; other chemistries require different tooling
- RawHash2's minimizer seeding mode is around 3X faster but slightly less accurate than the non-minimizer mode
- RawHash presets must be size-matched to the reference (-x sensitive for small genomes, -x fast for large genomes); mismatched presets degrade results
- FAST5 reads compressed with ONT VBZ compression require the VBZ HDF5 plugin to be readable

## Evidence

- [abstract] We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis.: "We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis."
- [abstract] Throughput and accuracy comparison: "When compared to the state-of-the-art techniques, UNCALLED and Sigmap, RawHash provides (i) 25.8× and 3.4× better average throughput and (ii) significantly better accuracy for large genomes,"
- [methods] Indexing example: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [methods] Mapping and chaining: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [readme] Sigmap index build command from README: [evidence span withheld — anchor status: external_doc]
- [readme] UNCALLED chemistry restriction: [evidence span withheld — anchor status: external_doc]
