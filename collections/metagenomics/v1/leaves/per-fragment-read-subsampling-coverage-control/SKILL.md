---
name: per-fragment-read-subsampling-coverage-control
description: Use when multiple high-coverage 10x Chromium Linked-Read libraries must
  be compared at matched parameters, or when an experiment needs to fix read coverage
  per DNA fragment (CR) while varying fragment-level parameters such as physical coverage
  (CF) or fragment length (Mu_F).
when_to_use_negative:
- Data are not barcode-based 10x Chromium Linked-Reads — fragment reconstruction relies
  on barcode-proximal read pairs and would be meaningless for PacBio/ONT long reads
- Libraries are already low-coverage at or below the target CR — subsampling cannot
  increase per-fragment coverage
- The goal is to simulate Linked-Reads from scratch rather than downsample existing
  sequenced libraries — use LRTK-SIM generation directly instead
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Long Ranger
  role: Aligns raw 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate
    marking prior to fragment reconstruction
- name: 10x Linked-Read sequencing
  role: Source technology whose barcode-linked read pairs provide the barcode-proximity
    signal used to infer original DNA fragments
- name: LRTK-SIM
  role: Defines the CR (coverage of short reads for each fragment) parameter used
    as the subsampling target, and validates that outputs of matching format are accepted
    by 10x official pipelines
  repo: https://github.com/zhanglu295/LRTK-SIM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:9bdd596b8e717336ee40e9cccce1ec60ebcea063ffad7c6faa1619ed1744b2fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - 10x Linked-Read sequencing
  - LRTK-SIM
  - Long Ranger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# per-fragment-read-subsampling-coverage-control

## Summary

A method for making heterogeneous high-coverage 10x Linked-Read libraries comparable by reconstructing the original input DNA fragments from barcode-proximal read pairs and randomly subsampling reads within each fragment to a fixed per-fragment short-read coverage (CR). It was used in the Zhang et al. 2019 GigaScience study of human diploid assembly with 10x Linked-Reads.

## When to use

Use when multiple high-coverage 10x Chromium Linked-Read libraries must be compared at matched parameters, or when an experiment needs to fix read coverage per DNA fragment (CR) while varying fragment-level parameters such as physical coverage (CF) or fragment length (Mu_F).

## When NOT to use

- Data are not barcode-based 10x Chromium Linked-Reads — fragment reconstruction relies on barcode-proximal read pairs and would be meaningless for PacBio/ONT long reads
- Libraries are already low-coverage at or below the target CR — subsampling cannot increase per-fragment coverage
- The goal is to simulate Linked-Reads from scratch rather than downsample existing sequenced libraries — use LRTK-SIM generation directly instead

## Inputs

- Raw 10x Chromium paired-end FASTQ files (R1/R2) from high-coverage Linked-Read libraries
- hg38 human reference genome sequence
- Long Ranger alignment outputs with barcode-aware duplicate marking
- Target per-fragment short-read coverage value (CR)

## Outputs

- Gzipped R1/R2 FASTQ pair of subsampled reads (e.g. X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz)
- Per-partition table of inferred fragment count, fragment length, and realized coverage

## How to apply

First align the raw 10x paired-end reads to the hg38 human reference genome with the Long Ranger pipeline, and remove PCR duplicates in a barcode-aware manner using Long Ranger's duplicate marking. Reconstruct the input DNA fragments by grouping barcode-proximal read pairs: terminate fragment extension at gaps >50 kb and retain only fragments supported by >=2 read pairs and >=2 kb in length; discard partitions (droplets) containing fewer than 3 inferred fragments. Then randomly subsample read pairs within each retained fragment until the target short-read coverage per fragment (CR) is met — CR is the same control parameter used by the LRTK-SIM simulator (line 9 of its config). The rationale is that assembly quality depends on CR independently of total sequencing coverage C, so equalizing CR lets libraries with different C, CF, or fragment length be compared fairly.

## Related tools

- **Long Ranger** (Aligns raw 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate marking prior to fragment reconstruction)
- **10x Linked-Read sequencing** (Source technology whose barcode-linked read pairs provide the barcode-proximity signal used to infer original DNA fragments)
- **LRTK-SIM** (Defines the CR (coverage of short reads for each fragment) parameter used as the subsampling target, and validates that outputs of matching format are accepted by 10x official pipelines) — https://github.com/zhanglu295/LRTK-SIM

## Examples

```
python LRTK-SIM.py ./diploid_config  # config files set line9: CR = target per-fragment short-read coverage
```

## Evaluation signals

- The gzipped FASTQ pair (R1/R2) exists and is well-formed for each subsampled library
- Realized per-fragment read coverage qualitatively matches the configured target CR (the paper reports no numeric tolerance)
- Only fragments supported by >=2 read pairs, >=2 kb in length, and within partitions of >=3 fragments are retained in the subsampled set
- Subsampled reads pass Long Ranger validation — LRTK-SIM output in this format was 'tested and accepted by 10X official pipelines Long Ranger and Supernova'
- Per-partition tabulation of fragment count, length, and realized coverage is produced and internally consistent with the FASTQ content

## Limitations

- The paper reports no numeric tolerance for CR matching, so acceptance is a qualitative match of realized CR to the target
- Only ∼20–40% of the original long fragment sequence is generally captured in each 10x partition due to partition-level amplification, bounding the reads available for subsampling
- Fragments spanning repetitive sequences can lose barcode specificity with extremely long fragments, degrading downstream assembly quality even after CR normalization
- LRTK-SIM config only supports SNV insertion into the reference for diploid simulation (only SNVs for this version)

## Evidence

- [methods] Alignment of raw reads and barcode-aware duplicate removal, followed by fragment inference and CR-targeted read subsampling: [evidence span withheld — anchor status: not_found]
- [methods] Subsampling procedure definition: [evidence span withheld — anchor status: not_found]
- [readme] CR as a controlled simulator parameter: "`line9`: `CR`, covergae of short reads for each fragment"
- [readme] Downstream pipeline acceptance of subsampled-format reads: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger)"
- [abstract] Scientific rationale for controlling CR: "We found that assembly quality could be improved by increasing the total sequencing coverage (C) and keeping physical coverage of DNA fragments (CF) or read coverage per fragment (CR) within broad"
