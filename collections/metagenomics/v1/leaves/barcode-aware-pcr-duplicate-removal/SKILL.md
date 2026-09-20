---
name: barcode-aware-pcr-duplicate-removal
description: Use when raw 10x Chromium Linked-Read paired-end FASTQs must be deduplicated
  before fragment inference or subsampling to a target CR — e.g., building comparable
  Linked-Read sets (Rid) with matched per-fragment read coverage across libraries.
when_to_use_negative:
- Input FASTQ lacks 10x barcodes (e.g., standard Illumina WGS, PacBio, or ONT reads)
  — barcode-aware marking is meaningless and position-based duplicate removal applies
  instead.
- Input is already aligned and duplicate-marked by Long Ranger (e.g., a Supernova-ready
  BAM); re-running duplicate removal adds nothing.
- The goal is variant calling or assembly directly on high-coverage libraries without
  needing matched per-fragment coverage (CR) across libraries.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0102
tools:
- name: Long Ranger
  role: Aligns raw 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate
    marking prior to fragment inference and CR-targeted subsampling
  repo: https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- name: LRTK-SIM
  role: Generates simulated 10x Linked-Read FASTQ pairs (tested and accepted by Long
    Ranger) that can exercise the duplicate-removal and fragment-inference workflow
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: 10x Chromium system
  role: Source of the barcode-partitioned libraries whose per-partition amplification
    creates the PCR duplicates that this skill removes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:47855dc1d76a62e08f6c584d09cf1cd43249a2cc5fa71e08d88286c595f23a47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Chromium system
  - LRTK-SIM
  - Long Ranger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# barcode-aware-pcr-duplicate-removal

## Summary

Removes PCR duplicates from 10x Chromium Linked-Read data using Long Ranger's barcode-aware duplicate marking after alignment to hg38, so that downstream fragment inference and subsampling to a target per-fragment read coverage (CR) reflect unique DNA molecules rather than partition-amplified copies.

## When to use

Use when raw 10x Chromium Linked-Read paired-end FASTQs must be deduplicated before fragment inference or subsampling to a target CR — e.g., building comparable Linked-Read sets (Rid) with matched per-fragment read coverage across libraries.

## When NOT to use

- Input FASTQ lacks 10x barcodes (e.g., standard Illumina WGS, PacBio, or ONT reads) — barcode-aware marking is meaningless and position-based duplicate removal applies instead.
- Input is already aligned and duplicate-marked by Long Ranger (e.g., a Supernova-ready BAM); re-running duplicate removal adds nothing.
- The goal is variant calling or assembly directly on high-coverage libraries without needing matched per-fragment coverage (CR) across libraries.

## Inputs

- Raw 10x Chromium Linked-Read paired-end FASTQ files (R1/R2, gzipped) carrying 10x barcodes (e.g., X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz)
- hg38 human reference genome sequence
- Long Ranger reference/organism data and Chromium chemistry settings

## Outputs

- hg38-aligned, barcode-aware duplicate-marked BAM produced by Long Ranger
- Downstream: set of inferred input DNA fragments (each >=2 read pairs, >=2 kb, terminated at >50 kb gaps) and a subsampled gzipped R1/R2 FASTQ pair meeting the target CR

## How to apply

First align the raw 10x paired-end R1/R2 FASTQ files to the hg38 human reference genome with the Long Ranger pipeline, then rely on Long Ranger's duplicate marking, which operates in a barcode-aware manner — duplicates are identified within the same 10x barcode/partition rather than by mapping position alone. This is essential because the Chromium system performs amplification within each partition and generally only ~20-40% of the original long fragment sequence is captured, so position-only deduplication would conflate independently barcoded molecules. After duplicate removal, reconstruct input DNA fragments by grouping barcode-proximal read pairs (extension terminates at gaps >50 kb; retain fragments supported by >=2 read pairs and >=2 kb in length), drop partitions with fewer than 3 fragments, and randomly subsample read pairs within each retained fragment until the target short-read coverage per fragment (CR) is met. The rationale is that unremoved duplicates would inflate per-fragment read coverage and bias the CR used to match libraries across the parameter space studied (C, CF, CR).

## Related tools

- **Long Ranger** (Aligns raw 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate marking prior to fragment inference and CR-targeted subsampling) — https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- **LRTK-SIM** (Generates simulated 10x Linked-Read FASTQ pairs (tested and accepted by Long Ranger) that can exercise the duplicate-removal and fragment-inference workflow) — https://github.com/zhanglu295/LRTK-SIM
- **10x Chromium system** (Source of the barcode-partitioned libraries whose per-partition amplification creates the PCR duplicates that this skill removes)

## Evaluation signals

- Duplicate marking is barcode-aware: flagged duplicates share the same 10x barcode/partition, not merely the same mapping position.
- Downstream fragment inference retains only fragments supported by >=2 read pairs and >=2 kb in length, and partitions with <3 fragments are dropped.
- Realized per-fragment read coverage qualitatively matches the configured CR (the paper reports no numeric tolerance, so acceptance is a qualitative match).
- The output gzipped R1/R2 FASTQ pair exists and is well-formed.
- Alignment and deduplication complete through the official Long Ranger pipeline, which is known to accept such FASTQ inputs without error.

## Limitations

- Per-partition amplification in the Chromium system captures generally only ~20-40% of the original long fragment sequence, so deduplicated data represent an amplified subset of each input fragment.
- No numeric tolerance is reported for matching realized per-fragment coverage to the target CR; acceptance is qualitative.
- Demonstrated on hg38 human reference and NA12878/NA24385 10x libraries; behavior on other genomes or chemistries is not established in this source.

## Evidence

- [intro] An alternative approach is offered by 10x Genomics Chromium system, which distributes the DNA preparation into millions of partitions: [evidence span withheld — anchor status: not_found]
- [discussion] the 10x Chromium system performs amplification in each partition, and generally only ∼20–40% of the original long fragment sequence can be captured: "the 10x Chromium system performs amplification in each partition, and generally only ∼20–40% of the original long fragment sequence can be captured"
- [abstract] We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets.: "We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets."
- [readme] The simulated data have been tested and accepted by 10X official pipelines [Long Ranger]: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger]"
- [readme] `line9`: `CR`, covergae of short reads for each fragment: "`line9`: `CR`, covergae of short reads for each fragment"
