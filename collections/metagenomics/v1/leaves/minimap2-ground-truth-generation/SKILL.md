---
name: minimap2-ground-truth-generation
description: 'Use when evaluating raw nanopore signal mapping tools against a trusted
  reference: minimap2 alignments of basecalled reads serve as ground truth to compute
  per-dataset precision/recall/F1 for raw-signal mappers.'
when_to_use_negative:
- When the goal is the raw-signal mapping itself rather than benchmarking — minimap2
  operates on basecalled sequences, not raw signals, and cannot replace the signal
  mappers being evaluated
- When no high-quality basecalled reads are available for the dataset, since the truth
  set cannot be generated from raw signals alone with this method
- When evaluating on r10.4.1 chemistry data — UNCALLED (whose PAF template and pafstats
  are reused) only supports legacy r9.4.1 data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Aligns basecalled reads to the reference genome to produce the ground-truth
    mapping set used for scoring raw-signal mapper outputs
  repo: https://github.com/lh3/minimap2
- name: RawHash
  role: Raw-signal mapper under evaluation; its PAF output is scored against the minimap2
    ground truth
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Raw-signal mapper under evaluation; also supplies the PAF output template
    and the pafstats scoring utility
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Raw-signal mapper under evaluation; its PAF output is scored against the minimap2
    ground truth
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:17701ad06e4ee4f78156b987d2405e1afa1512da012be77ea58fed806dca1e97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
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
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# minimap2-ground-truth-generation

## Summary

Generates ground-truth basecalled-read alignments with minimap2 to score raw-signal mappers (RawHash/RawHash2, UNCALLED, Sigmap) on precision, recall, and error rate using UNCALLED's pafstats utility.

## When to use

Use when evaluating raw nanopore signal mapping tools against a trusted reference: minimap2 alignments of basecalled reads serve as ground truth to compute per-dataset precision/recall/F1 for raw-signal mappers.

## When NOT to use

- When the goal is the raw-signal mapping itself rather than benchmarking — minimap2 operates on basecalled sequences, not raw signals, and cannot replace the signal mappers being evaluated
- When no high-quality basecalled reads are available for the dataset, since the truth set cannot be generated from raw signals alone with this method
- When evaluating on r10.4.1 chemistry data — UNCALLED (whose PAF template and pafstats are reused) only supports legacy r9.4.1 data

## Inputs

- Basecalled nanopore reads (FASTQ/FASTA) derived from the raw FAST5/POD5/SLOW5/BLOW5 signal files
- Reference genome FASTA (e.g., SARS-CoV-2, yeast, green algae, human NA12878 references)
- PAF mapping outputs from RawHash, UNCALLED, and Sigmap in the UNCALLED-style PAF template
- UNCALLED pafstats utility for PAF scoring

## Outputs

- Ground-truth alignment files (SAM/PAF) from minimap2 per dataset
- Per-dataset, per-tool precision, recall, and error-rate scores from pafstats
- Aggregated accuracy_table.csv suitable for a Table 2-style comparison

## How to apply

For each of the five evaluation datasets (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, and the MinION contamination FAST5 files), basecall the raw signals and align the basecalled reads to the same reference genome with minimap2 using the map-ont preset to produce ground-truth alignments. Map the same raw signals with RawHash, UNCALLED, and Sigmap (default parameters), writing each tool's output in PAF format following the UNCALLED-style PAF template. Score each raw mapper's PAF against the minimap2 ground truth with UNCALLED's pafstats utility to obtain error rate, precision, and recall per dataset and tool. The rationale is that basecalled-read alignments from the mature minimap2 mapper provide a stable mapping truth set, so relative differences among raw-signal mappers reflect genuine accuracy differences rather than reference inconsistencies. Aggregate the per-tool, per-dataset metrics into an accuracy_table.csv that must reproduce the paper's qualitative ordering: RawHash significantly better for large genomes (human) and on par for smaller genomes.

## Related tools

- **Minimap2** (Aligns basecalled reads to the reference genome to produce the ground-truth mapping set used for scoring raw-signal mapper outputs) — https://github.com/lh3/minimap2
- **RawHash** (Raw-signal mapper under evaluation; its PAF output is scored against the minimap2 ground truth) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Raw-signal mapper under evaluation; also supplies the PAF output template and the pafstats scoring utility) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Raw-signal mapper under evaluation; its PAF output is scored against the minimap2 ground truth) — https://github.com/haowenz/sigmap

## Examples

```
minimap2 -ax map-ont ref.fa basecalled_reads.fq > ground_truth.sam
```

## Evaluation signals

- Every PAF output from RawHash, UNCALLED, and Sigmap is emitted in the UNCALLED-style PAF template so pafstats can parse it without format errors
- Per-dataset precision, recall, and error-rate values are present for all three tools across datasets D1-D5
- The accuracy_table.csv reproduces the paper's qualitative ordering: RawHash significantly better on large genomes (human) and on par with UNCALLED and Sigmap on smaller genomes (yeast, green algae, SARS-CoV-2)
- Ground-truth alignments are generated against the exact same reference FASTA used to build the raw mappers' indexes (including the same k-mer model, e.g., legacy_r9.4_180mv_450bps_6mer/template_median68pA.model for RawHash)

## Limitations

- Ground truth is minimap2 on basecalled reads, so basecalling errors propagate into the truth set; a mapper disagreeing with minimap2 is not necessarily wrong
- UNCALLED and its PAF template/pafstats utility are only applicable to legacy r9.4.1 data, limiting the chemistry range of the comparison
- For the contamination dataset, mapping-based contamination detection and relative abundance estimates are derived downstream of the scored alignments, adding an interpretation layer beyond raw precision/recall

## Evidence

- [abstract] RawHash achieves the best accuracy in precision, recall, and F1 compared to UNCALLED and Sigmap when mapping reads to large genomes: [evidence span withheld — anchor status: not_found]
- [methods] UNCALLED-style PAF template: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [readme] minimap2 map-ont preset: [evidence span withheld — anchor status: external_doc]
- [abstract] Real-time analysis scope: "Our evaluations show that RawHash is the only tool that can provide high accuracy and high throughput for analyzing large genomes in real-time."
