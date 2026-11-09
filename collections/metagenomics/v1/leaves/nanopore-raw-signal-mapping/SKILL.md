---
name: nanopore-raw-signal-mapping
description: Use when the input is raw nanopore signal data (FAST5, POD5, SLOW5, or
  BLOW5) from legacy r9.4 chemistry that must be mapped to a reference genome in real
  time, especially for large genomes such as human NA12878.
when_to_use_negative:
- The input is already basecalled sequence data (FASTQ/BAM) — use a base-level mapper
  such as minimap2 instead of raw-signal mapping
- The reads are from r10.4.1 chemistry — UNCALLED can only be applied to legacy r9.4.1
  data, and the legacy 6-mer pore model used here is r9.4-specific (use ReadFish or
  ONT builtin adaptive sampling instead)
- No reference genome is available and all-vs-all overlaps are needed for de novo
  assembly — use Rawsamble rather than reference-based raw signal mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash
  role: Primary raw-signal mapper; indexes the reference into hash values from expected
    signal representations and maps FAST5/POD5/SLOW5/BLOW5 signals via hash-based
    similarity search and seed-match chaining
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: More sensitive and faster raw signal mapping mechanism with substantial improvements
    over RawHash; invoked as the rawhash2 binary for indexing and mapping
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline raw-signal mapper for comparison; its PAF template defines the output
    format and its pafstats utility scores precision/recall and throughput
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming raw-signal mapper for comparison; indexes the reference
    using ONT pore models before mapping
  repo: https://github.com/haowenz/sigmap
- name: Minimap2
  role: Aligns basecalled reads to produce the ground-truth alignments used to score
    raw-signal mapping accuracy
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:aab77f040e7a0c0986db769a76584aa672e5b6ebfd54819e66cadf821736f339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - Minimap2
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# nanopore-raw-signal-mapping

## Summary

Maps raw nanopore electrical signals (not basecalled reads) directly to a reference genome using hash-based signal-to-hash search with RawHash/RawHash2, benchmarked against signal mappers UNCALLED and Sigmap, with accuracy scored against minimap2 basecalled-read ground truth.

## When to use

Use when the input is raw nanopore signal data (FAST5, POD5, SLOW5, or BLOW5) from legacy r9.4 chemistry that must be mapped to a reference genome in real time, especially for large genomes such as human NA12878.

## When NOT to use

- The input is already basecalled sequence data (FASTQ/BAM) — use a base-level mapper such as minimap2 instead of raw-signal mapping
- The reads are from r10.4.1 chemistry — UNCALLED can only be applied to legacy r9.4.1 data, and the legacy 6-mer pore model used here is r9.4-specific (use ReadFish or ONT builtin adaptive sampling instead)
- No reference genome is available and all-vs-all overlaps are needed for de novo assembly — use Rawsamble rather than reference-based raw signal mapping

## Inputs

- Raw nanopore signal files in FAST5 format (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94 datasets; POD5/SLOW5/BLOW5 also supported by RawHash)
- Reference genome in FASTA format (e.g., SARS-CoV-2, yeast, green algae, human genome)
- ONT k-mer pore model file (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- Basecalled FASTQ/FASTA reads from the same datasets for minimap2 ground-truth alignment

## Outputs

- mapping.paf — per-tool raw-signal alignments in the UNCALLED-style PAF template
- ref.ind — RawHash reference index
- Per-tool, per-dataset precision, recall, F1, error rate, and throughput table (accuracy_table.csv equivalent from pafstats scoring against minimap2 ground truth)

## How to apply

Build a reference index with rawhash2 ('rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta'), selecting the genome-size preset from the README: -x viral for the SARS-CoV-2 set (d1_sars-cov-2_r94), -x sensitive for small-genome sets (d3_yeast_r94, d4_green_algae_r94), and -x fast for the >500M human NA12878 set. Map the raw signals with RawHash, UNCALLED, and Sigmap under identical conditions (32 threads, default raw-signal mapping settings), writing each tool's output in the UNCALLED-style PAF template to mapping.paf. RawHash works by generating hash values from quantized raw signals and querying a hash table built from the expected signal representation of the reference genome, then chaining over the seed matches. Establish ground truth by aligning basecalled reads with minimap2, then score each tool's PAF output with UNCALLED's pafstats utility to obtain error rate, precision, recall, and throughput (bases/second). The rationale is that RawHash's quantization maps signals from the same DNA content to identical hash values, yielding higher accuracy and throughput (25.8x and 3.4x average over UNCALLED and Sigmap) than DTW-style or alternative hash-based signal mappers, particularly on large genomes.

## Related tools

- **RawHash** (Primary raw-signal mapper; indexes the reference into hash values from expected signal representations and maps FAST5/POD5/SLOW5/BLOW5 signals via hash-based similarity search and seed-match chaining) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (More sensitive and faster raw signal mapping mechanism with substantial improvements over RawHash; invoked as the rawhash2 binary for indexing and mapping) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline raw-signal mapper for comparison; its PAF template defines the output format and its pafstats utility scores precision/recall and throughput) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming raw-signal mapper for comparison; indexes the reference using ONT pore models before mapping) — https://github.com/haowenz/sigmap
- **Minimap2** (Aligns basecalled reads to produce the ground-truth alignments used to score raw-signal mapping accuracy) — https://github.com/lh3/minimap2

## Examples

```
rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta
```

## Evaluation signals

- Per-tool PAF outputs for all five datasets (D1-D5) score non-trivially against minimap2 ground truth via UNCALLED's pafstats, with precision, recall, and error rate present for RawHash, UNCALLED, and Sigmap
- The reconstructed accuracy_table.csv reproduces the paper's qualitative ordering: RawHash achieves significantly better precision/recall/F1 than UNCALLED and Sigmap on large genomes (human) and is on par for smaller genomes
- Measured average throughput ratios across D1-D5 match the reported 25.8x vs UNCALLED and 3.4x vs Sigmap, and Sigmap's human-genome (D5) throughput falls below 1x the nanopore sequencing rate
- Index construction completes with the legacy r9.4 6-mer pore model and 32 threads producing ref.ind, and mapping uses the correct genome-size preset per dataset (-x viral, -x sensitive, -x fast)
- All three tools run under identical conditions (same datasets, references, 32 threads) so throughput comparisons are valid

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data; the pipeline's pore model and datasets are r9.4-specific
- Sigmap cannot perform real-time genome analysis for large genomes, achieving only 0.7x and 0.6x the throughput of a nanopore for human genome mapping and relative abundance estimation
- FAST5 reads compressed with VBZ compression require the nanoporetech/vbz_compression HDF5 plugin to be installed before reading
- Mapping accuracy is approximate (PAF without base-level alignment) and is validated only against minimap2 basecalled-read alignments as ground truth

## Evidence

- [methods] RawHash can be used to map reads from FAST5, POD5, SLOW5, or BLOW5 files to a reference genome in sequence format.: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [abstract] We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis.: "We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis."
- [methods] To map the raw signal to a reference genome, RawHash performs chaining over the seed matches.: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [readme] A read mapper which rapidly aligns raw nanopore signal to DNA references: [evidence span withheld — anchor status: external_doc]
- [abstract] RawHash provides (i) 25.8× and 3.4× better average throughput and (ii) significantly better accuracy for large genomes, respectively.: "RawHash provides (i) 25.8× and 3.4× better average throughput and (ii) significantly better accuracy for large genomes, respectively."
