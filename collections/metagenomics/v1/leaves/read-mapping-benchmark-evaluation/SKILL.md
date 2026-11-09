---
name: read-mapping-benchmark-evaluation
description: Use when comparing mapping accuracy or throughput of raw-signal nanopore
  mappers against a reference genome, or when validating that a new mapper's precision/recall
  matches published orderings on datasets spanning small to large genomes.
when_to_use_negative:
- The input is already basecalled sequence data (FASTQ) — minimap2 or standard read
  mappers should be used directly instead of raw-signal mappers
- The goal is all-vs-all signal overlap for de novo assembly without a reference —
  use Rawsamble instead of read-mapping benchmarking
- The data is r10.4.1 nanopore reads — UNCALLED only applies to legacy r9.4.1 data
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Primary hash-based raw-signal mapper under evaluation; builds reference index
    and maps raw signals in real-time
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline raw-signal mapper for accuracy comparison; its PAF template and pafstats
    utility are used for output formatting and ground-truth scoring
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming raw-signal mapper for accuracy comparison; indexed with
    ONT pore models
  repo: https://github.com/haowenz/sigmap
- name: Minimap2
  role: Generates basecalled-read alignment ground truth against the reference for
    scoring tool PAFs
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:34db37ee5ebf164ef15c9d5fad5f48d9b812410a6fb0ba432263fce283a449a1
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# read-mapping-benchmark-evaluation

## Summary

Benchmarks raw nanopore signal-to-reference mappers (RawHash, UNCALLED, Sigmap) on precision, recall, F1, and error rate across datasets, using minimap2 basecalled-read alignments as ground truth. Enables quantitative comparison of mapping accuracy and throughput for read mapping, relative abundance estimation, and contamination analysis.

## When to use

Use when comparing mapping accuracy or throughput of raw-signal nanopore mappers against a reference genome, or when validating that a new mapper's precision/recall matches published orderings on datasets spanning small to large genomes.

## When NOT to use

- The input is already basecalled sequence data (FASTQ) — minimap2 or standard read mappers should be used directly instead of raw-signal mappers
- The goal is all-vs-all signal overlap for de novo assembly without a reference — use Rawsamble instead of read-mapping benchmarking
- The data is r10.4.1 nanopore reads — UNCALLED only applies to legacy r9.4.1 data

## Inputs

- Raw nanopore signal files in FAST5, POD5, SLOW5, or BLOW5 format (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, contamination MinION FAST5)
- Reference genome in FASTA format (e.g., ref.fasta)
- ONT k-mer pore model file (e.g., legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- Basecalled reads in FASTQ/FASTA for minimap2 ground-truth alignments

## Outputs

- PAF mapping output per tool and dataset (mapping.paf, UNCALLED-style template)
- Per-tool, per-dataset precision, recall, F1, and error rate metrics (accuracy_table.csv)
- Relative abundance and contamination detection estimates for the contamination dataset
- Aggregated throughput comparison table (Table 2-equivalent)

## How to apply

Build a reference index for each signal mapper (e.g., RawHash: 'rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta'; Sigmap uses a similar ONT pore model from extern/kmer_models). Map each dataset (d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, and MinION contamination FAST5 files) with RawHash, UNCALLED, and Sigmap using default parameters, writing PAF outputs following the UNCALLED PAF template. Generate ground truth by mapping basecalled reads with minimap2 ('minimap2 -ax map-ont ref.fa ont.fq.gz'). Score each tool's PAF against the minimap2 ground truth using UNCALLED's pafstats utility to obtain error rate, precision, and recall per dataset/tool. For the contamination dataset, additionally derive contamination detection and relative abundance estimates from mapped reads, then aggregate per-tool, per-dataset metrics and throughput into a comparison table. Decision point: expect RawHash to significantly outperform UNCALLED and Sigmap on large genomes (human) while remaining on par for small genomes (SARS-CoV-2, yeast).

## Related tools

- **RawHash** (Primary hash-based raw-signal mapper under evaluation; builds reference index and maps raw signals in real-time) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline raw-signal mapper for accuracy comparison; its PAF template and pafstats utility are used for output formatting and ground-truth scoring) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming raw-signal mapper for accuracy comparison; indexed with ONT pore models) — https://github.com/haowenz/sigmap
- **Minimap2** (Generates basecalled-read alignment ground truth against the reference for scoring tool PAFs) — https://github.com/lh3/minimap2

## Examples

```
rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta
```

## Evaluation signals

- accuracy_table.csv contains per-dataset precision, recall, and error-rate values for all three tools (RawHash, UNCALLED, Sigmap)
- Qualitative ordering reproduces the paper: RawHash shows significantly better accuracy for large genomes (human NA12878) than UNCALLED and Sigmap
- RawHash and baselines are on par in accuracy for smaller genomes (SARS-CoV-2, yeast, green algae)
- Average throughput ordering matches reported ratios: RawHash 25.8x faster than UNCALLED and 3.4x faster than Sigmap
- Each tool's PAF output follows the UNCALLED-style PAF template and is scoreable by pafstats against minimap2 ground truth

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, limiting baseline comparisons on newer chemistries
- Accuracy advantage of RawHash is genome-size dependent: it holds for large genomes (human) but not for smaller genomes
- RawHash requires VBZ-compressed FAST5 files to be handled with the vbz_compression HDF5 plugin
- Ground truth is itself an approximation: minimap2 alignments of basecalled reads, so precision/recall are relative to that reference standard

## Evidence

- [results] RawHash achieves the best accuracy in precision, recall, and F1 compared to UNCALLED and Sigmap when mapping reads to large genomes: [evidence span withheld — anchor status: not_found]
- [abstract] throughput claim: "RawHash provides (i) 25.8× and 3.4× better average throughput and (ii) significantly better accuracy for large genomes"
- [methods] indexing example: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [methods] paf template: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [readme] sigmap usage: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 preset: [evidence span withheld — anchor status: external_doc]
