---
name: short-read-alignment-to-reference-genome
description: Use when raw 10x Chromium paired-end FASTQ reads must be mapped to hg38
  as a prerequisite for barcode-aware duplicate removal, fragment inference, and read
  coverage (CR) subsampling.
when_to_use_negative:
- The goal is de novo assembly directly from the raw data without subsampling to matched
  CR across libraries — use Supernova2 on the full library instead.
- Input is long-read PacBio or Oxford Nanopore (ONT) data; Long Ranger and this fragment-inference
  logic are specific to 10x Linked-Read short reads.
- The aim is simulation of new Linked-Reads rather than processing real libraries
  — use LRTK-SIM directly with its config parameters (CF, CR, Mu_F, SR).
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Long Ranger
  role: Aligns 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate
    marking, providing the aligned input for fragment inference
  repo: https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- name: 10x Linked-Read sequencing
  role: Source sequencing technology producing the barcode-proximal paired-end reads
    that are aligned
- name: LRTK-SIM
  role: Simulator whose output FASTQ files have been tested and accepted by the Long
    Ranger pipeline; used to generate comparable Linked-Read sets at target CR
  repo: https://github.com/zhanglu295/LRTK-SIM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:19a32967be7d61a2d509e2a0bb64fbb79de35a212c883ef32c274412bc6f42e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Linked-Read sequencing
  - LRTK-SIM
  - Long Ranger
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# short-read-alignment-to-reference-genome

## Summary

Align 10x Linked-Read paired-end reads to the hg38 human reference with the Long Ranger pipeline as the first step of the LRTK-SIM subsampling workflow, preserving barcode information so PCR duplicates can be removed in a barcode-aware manner before DNA fragments are inferred.

## When to use

Use when raw 10x Chromium paired-end FASTQ reads must be mapped to hg38 as a prerequisite for barcode-aware duplicate removal, fragment inference, and read coverage (CR) subsampling.

## When NOT to use

- The goal is de novo assembly directly from the raw data without subsampling to matched CR across libraries — use Supernova2 on the full library instead.
- Input is long-read PacBio or Oxford Nanopore (ONT) data; Long Ranger and this fragment-inference logic are specific to 10x Linked-Read short reads.
- The aim is simulation of new Linked-Reads rather than processing real libraries — use LRTK-SIM directly with its config parameters (CF, CR, Mu_F, SR).

## Inputs

- Raw 10x Linked-Read paired-end FASTQ reads (R1/R2, gzip) from NA12878 or NA24385 libraries
- hg38 human reference genome sequence
- 10x Chromium sample index / barcode structure implicit in the FASTQ headers

## Outputs

- Position-sorted BAM of reads aligned to hg38 with barcode-aware PCR duplicate marking
- Inferred input DNA fragments (>=2 read pairs, >=2 kb, terminated at >50 kb gaps)
- Subsampled gzipped R1/R2 FASTQ pairs at the target CR per fragment
- Per-partition table of fragment count, fragment length, and realized coverage

## How to apply

Run the 10x Long Ranger pipeline to align the raw paired-end reads against the hg38 human reference genome; alignment with Long Ranger (rather than a generic aligner) keeps molecule barcodes attached so PCR duplicates can subsequently be marked in a barcode-aware manner. From the aligned BAM, group barcode-proximal read pairs into input DNA fragments, terminating extension at gaps >50 kb and retaining only fragments supported by >=2 read pairs and >=2 kb in length. Drop partitions (GEMs) containing fewer than 3 inferred fragments, then randomly subsample read pairs within each retained fragment until the target short-read coverage per fragment (CR) is met. This aligned-and-deduplicated step is what makes high-coverage libraries comparable across the eight 10x libraries before assembly evaluation.

## Related tools

- **Long Ranger** (Aligns 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate marking, providing the aligned input for fragment inference) — https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- **10x Linked-Read sequencing** (Source sequencing technology producing the barcode-proximal paired-end reads that are aligned)
- **LRTK-SIM** (Simulator whose output FASTQ files have been tested and accepted by the Long Ranger pipeline; used to generate comparable Linked-Read sets at target CR) — https://github.com/zhanglu295/LRTK-SIM

## Examples

```
longranger align --id=lib1 --fastqs=./fastqs --reference=hg38 --chemistry=SC3PATHWAYHAPLOTYPES
```

## Evaluation signals

- The output gzipped R1/R2 FASTQ pair exists and is well-formed.
- Realized per-fragment short-read coverage matches the configured target CR (the paper reports no numeric tolerance, so acceptance is a qualitative match).
- Every retained fragment has >=2 read pairs and length >=2 kb, with extension terminated at gaps >50 kb.
- No partition with fewer than 3 inferred fragments remains in the output.
- LRTK-SIM-style output FASTQs are accepted by the official 10x Long Ranger and Supernova pipelines, as the authors report for their simulated data.

## Limitations

- The paper reports no numeric tolerance for the match between realized CR and the target CR, so validation of subsampling depth is qualitative.
- Only SNV-based diploid reference generation is supported in the LRTK-SIM companion workflow (gen_fasta.py inserts 'only SNVs for this version').
- Barcode specificity can be lost for fragments spanning repetitive sequences, and extremely long fragments were found to be detrimental for assembly quality.

## Evidence

- [methods] Reads were aligned to hg38, PCR duplicates removed barcode-aware in Long Ranger, fragments inferred from barcode-proximal read pairs (>50 kb gap terminates, >=2 read pairs, >=2 kb): [evidence span withheld — anchor status: not_found]
- [methods] partitions with <3 fragments dropped, and reads subsampled per fragment to a target CR: [evidence span withheld — anchor status: not_found]
- [readme] The simulated data have been tested and accepted by 10X official pipelines [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger) and [Supernova]: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger)"
- [readme] LRTK-SIM generates one folder for each library named after lib_X (X is the name of configuration file): "LRTK-SIM generates one folder for each library named after lib_X (X is the name of configuration file)"
