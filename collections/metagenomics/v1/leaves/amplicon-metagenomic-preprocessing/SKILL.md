---
name: amplicon-metagenomic-preprocessing
description: Use when raw amplicon FASTQ reads (454 or Illumina 16S/ITS) need primer
  trimming plus adaptive quality filtering before OTU clustering, and reads exceeding
  a maximum expected error (EE) budget must be discarded.
when_to_use_negative:
- Input reads are already primer-trimmed and quality-filtered (e.g., downstream of
  MICCA's trim/filter, feeding OTU clustering or chimera detection directly)
- The goal is UPARSE-style stringent quality filtering, which truncates reads at the
  first base below a quality threshold and discards a large fraction of sequence data
  — MICCA's EE-based filtering is deliberately more flexible
- Data are shotgun metagenomic reads rather than targeted conserved amplicons (16S
  rRNA, ITS, 18S, 28S), which is MICCA's declared scope
edam_operation: http://edamontology.org/operation_3188
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: MICCA
  role: Pipeline wrapper providing the 'trim' and 'filter' commands that implement
    primer trimming and EE-based quality filtering
  repo: https://github.com/compmetagen/micca
- name: Cutadapt
  role: Removes primer sequences from raw reads in the MICCA 'trim' command, using
    IUPAC degenerate primer motifs
- name: VSEARCH
  role: Wrapped by MICCA 'filter' to perform the read-quality filtering pass alongside
    sickle
- name: sickle
  role: Sliding-window adaptive quality trimming tool used in the filter step; window
    is 0.1x read length, cuts 5'/3' ends by average quality, discards reads below
    minimum length
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:3ab558cd96fadb81a7b6fc613bf744a4b256a1c839ed7091a3b82029e5a710f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - Cutadapt
  - MICCA
  - VSEARCH
  - sickle
  edam_operation: http://edamontology.org/operation_3188
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# amplicon-metagenomic-preprocessing

## Summary

MICCA's raw-read preprocessing skill for targeted metagenomic (amplicon) data: primer removal followed by sliding-window adaptive quality filtering that discards reads whose expected error (EE) exceeds the allowed maximum.

## When to use

Use when raw amplicon FASTQ reads (454 or Illumina 16S/ITS) need primer trimming plus adaptive quality filtering before OTU clustering, and reads exceeding a maximum expected error (EE) budget must be discarded.

## When NOT to use

- Input reads are already primer-trimmed and quality-filtered (e.g., downstream of MICCA's trim/filter, feeding OTU clustering or chimera detection directly)
- The goal is UPARSE-style stringent quality filtering, which truncates reads at the first base below a quality threshold and discards a large fraction of sequence data — MICCA's EE-based filtering is deliberately more flexible
- Data are shotgun metagenomic reads rather than targeted conserved amplicons (16S rRNA, ITS, 18S, 28S), which is MICCA's declared scope

## Inputs

- Raw amplicon FASTQ file (e.g., simulated 16S-10 454 reads with realistic error model, from ftp://ftp.fmach.it/metagenomics/micca/scirep/)
- Primer sequences (IUPAC degenerate motifs) for Cutadapt-based trimming
- Quality-score encoding choice (sanger, Illumina, or Solexa) for sickle

## Outputs

- Single filtered-reads FASTQ file (filtered_reads.fastq) containing only surviving reads
- Set of discarded low-quality reads (those exceeding max expected error EE or minimum length)

## How to apply

First run MICCA's 'trim' command, which wraps Cutadapt (doi: 10.14806/ej.17.1.200), to remove primer sequences from raw reads, exploiting Cutadapt's support for IUPAC degenerate primer motifs. Then run MICCA's 'filter' command, which wraps VSEARCH (doi: 10.7717/peerj.2584) and sickle, applying sliding-window adaptive quality trimming, N-trimming, and minimum/maximum length filtering in a single pass. Sickle slides a window whose length is 0.1 times the read length across the quality values, cutting where average quality drops below the threshold; remaining reads shorter than the minimum length threshold are discarded entirely. The decisive discard criterion is MICCA's maximum allowed expected error (EE): reads whose expected error exceeds the allowed maximum are removed as low quality. Surviving reads are written to the single output FASTQ filtered_reads.fastq, which then feeds dereplication and OTU clustering.

## Related tools

- **MICCA** (Pipeline wrapper providing the 'trim' and 'filter' commands that implement primer trimming and EE-based quality filtering) — https://github.com/compmetagen/micca
- **Cutadapt** (Removes primer sequences from raw reads in the MICCA 'trim' command, using IUPAC degenerate primer motifs)
- **VSEARCH** (Wrapped by MICCA 'filter' to perform the read-quality filtering pass alongside sickle)
- **sickle** (Sliding-window adaptive quality trimming tool used in the filter step; window is 0.1x read length, cuts 5'/3' ends by average quality, discards reads below minimum length) — https://github.com/najoshi/sickle

## Examples

```
sickle se -f input_file.fastq -t sanger -o trimmed_output_file.fastq -q 33 -l 40
```

## Evaluation signals

- Output is a single FASTQ (filtered_reads.fastq) whose records contain no primer sequences (Cutadapt motifs no longer match read starts)
- Every surviving read satisfies the minimum length threshold and its expected error does not exceed the maximum allowed EE
- Number of reads discarded should be comparable to QIIME's (the article notes 'the number of reads discarded by MICCA and QIIME are very similar'), and far less loss than UPARSE (which kept only 42.3% on 16S-10)
- Sickle's trimming point should coincide with the first sliding-window drop in average quality below threshold, not with the first low-quality base as in UPARSE truncation
- Reads removed entirely (not trimmed) correspond to length-threshold or EE-threshold failures, not merely to local quality dips

## Limitations

- Sickle's Solexa quality setting is only an approximation because the conversion is a non-linear transformation
- Sickle removes the 2nd FASTQ record header on the '+' line and replaces it with just '+', altering record formatting
- Combined stringent quality truncation plus a minimum read length requirement (the UPARSE behaviour this skill avoids) causes loss of a large fraction of sequence data; conversely, EE-based filtering's flexibility depends on calibrating the EE maximum
- The article notes no official, well-studied benchmark datasets exist, so validation relied on simulated data with known OTU composition (e.g., 16S-10 with 20% chimeras)

## Evidence

- [intro] EE-based discard criterion: "filters low quality sequences according to the maximum allowed expected error (EE)"
- [readme] README restates EE filtering as a key MICCA feature: "filters low quality sequences according to the maximum allowed **expected
  error (EE) rate** %;"
- [readme] Sickle sliding-window mechanism: "Sickle is a tool that uses sliding windows along with quality and
length thresholds to determine when quality is sufficiently low to
trim the 3'-end of reads"
- [readme] Cutadapt is the trim-command wrapper: "Cutadapt (doi: 10.14806/ej.17.1.200) used in the ``trim`` command"
- [results] Contrast with UPARSE truncation strategy: "the stringent quality filtering approach used by UPARSE4, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold"
- [discussion] MICCA's filtering is more read-efficient: "UPARSE, which implements a stringent quality filtering algorithm at the cost of discarding a large fraction of the sequence data"
- [methods] Source dataset for the workflow: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
