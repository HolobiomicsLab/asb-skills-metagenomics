---
name: metagenomic-read-length-and-n-trimming
description: Use when raw or primer-trimmed amplicon FASTQ reads (454 or Illumina)
  show deteriorating 3'-end quality or embedded Ns and must be trimmed and length-filtered
  before dereplication and OTU clustering.
when_to_use_negative:
- Input is not raw read data — e.g. reads already dereplicated, an OTU table, or a
  taxonomic classification, which need no per-read trimming
- The analysis requires UPARSE-style stringent truncation at the first base below
  a quality threshold and can tolerate discarding a large fraction of sequence data
- 'Reads are untrimmed at the primer level: run primer removal (MICCA ``trim``/Cutadapt
  with IUPAC degenerate motifs) first, since filter does not remove primer sequences'
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: MICCA
  role: Pipeline whose ``filter`` command exposes sliding-window adaptive trimming,
    N-trimming, and length filtering in one pass
  repo: https://github.com/compmetagen/micca
- name: sickle
  role: Sliding-window, adaptive, quality-based trimming tool wrapped by MICCA's ``filter``;
    performs the 5'/3' window cuts and discards reads below the length threshold
  repo: https://github.com/najoshi/sickle
- name: VSEARCH
  role: Third-party engine used by MICCA's ``filter`` command (should be cited when
    used)
  repo: https://github.com/compmetagen/micca
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:564c28a7cfdaecfd231ba6c820a7915d98285731526fa0b98062230945a9fafc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - Cutadapt
  - MICCA
  - VSEARCH
  - sickle
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# metagenomic-read-length-and-n-trimming

## Summary

Applies sliding-window adaptive quality trimming, N-truncation, and minimum/maximum length filtering to amplicon FASTQ reads, as implemented by MICCA's ``filter`` command wrapping VSEARCH and sickle, so that low-quality tails and N-containing reads do not contaminate OTU inference.

## When to use

Use when raw or primer-trimmed amplicon FASTQ reads (454 or Illumina) show deteriorating 3'-end quality or embedded Ns and must be trimmed and length-filtered before dereplication and OTU clustering.

## When NOT to use

- Input is not raw read data — e.g. reads already dereplicated, an OTU table, or a taxonomic classification, which need no per-read trimming
- The analysis requires UPARSE-style stringent truncation at the first base below a quality threshold and can tolerate discarding a large fraction of sequence data
- Reads are untrimmed at the primer level: run primer removal (MICCA ``trim``/Cutadapt with IUPAC degenerate motifs) first, since filter does not remove primer sequences

## Inputs

- Single-end amplicon FASTQ file of primer-trimmed reads (e.g. Roche 454 16S-10 reads or Illumina MiSeq/HiSeq reads)
- Quality-score encoding specification (Sanger, Solexa, or Illumina/CASAVA type)
- Quality threshold, minimum/maximum read length, and N-truncation flag

## Outputs

- Single trimmed and filtered FASTQ file (e.g. filtered_reads.fastq) containing reads passing length and quality filters

## How to apply

Run MICCA's ``filter`` command on the primer-trimmed FASTQ (e.g. output of ``micca trim``/Cutadapt). The wrapped sickle algorithm slides a window of 0.1 times the read length (or the full read if this is <1) across quality values, cutting the 5'-end where average quality rises above the threshold and the 3'-end where it drops below, and discards any read whose remaining length falls under the minimum length threshold. Enable truncation of reads at the first N position and set an appropriate Sanger/Illumina quality encoding. Rationale: unlike UPARSE's truncation at the first low-quality base—which combined with a minimum read length loses a large fraction of data—sliding-window adaptive trimming retains more reads; final quality acceptance is judged by the maximum allowed expected error (EE) rate. Write surviving reads to a single output FASTQ such as filtered_reads.fastq.

## Related tools

- **MICCA** (Pipeline whose ``filter`` command exposes sliding-window adaptive trimming, N-trimming, and length filtering in one pass) — https://github.com/compmetagen/micca
- **sickle** (Sliding-window, adaptive, quality-based trimming tool wrapped by MICCA's ``filter``; performs the 5'/3' window cuts and discards reads below the length threshold) — https://github.com/najoshi/sickle
- **VSEARCH** (Third-party engine used by MICCA's ``filter`` command (should be cited when used)) — https://github.com/compmetagen/micca

## Examples

```
sickle se -f input_file.fastq -t sanger -o trimmed_output_file.fastq -x -n
```

## Evaluation signals

- Output FASTQ contains no read shorter than the configured minimum length threshold and no read with Ns if N-truncation was enabled
- Every surviving read's window-trimmed quality profile meets the maximum allowed expected error (EE) rate criterion
- Read count after filtering is comparable to QIIME's discard rate (MICCA and QIIME discard very similar numbers of reads), and substantially higher retention than UPARSE's truncation-based filtering
- Downstream OTU estimation on benchmark data (e.g. 16S-10) converges toward ~77% of the real OTU number, MICCA's reported performance
- 3'-end quality distribution of the trimmed FASTQ no longer shows the deteriorating quality tail described for modern sequencing reads

## Limitations

- Sickle's Solexa quality setting is an approximation; the actual conversion is a non-linear transformation and only the end approximation is close
- Sliding-window trimming can still discard reads entirely when the remaining sequence falls below the minimum length threshold (or replaces them with an 'N' record in paired mode)
- No official, well-studied benchmark datasets exist, so the accuracy and reliability of the filtering step are hard to assess independently of simulated data

## Evidence

- [readme] Sickle is a tool that uses sliding windows along with quality and length thresholds to determine when quality is sufficiently low to trim the 3'-end of reads: [evidence span withheld — anchor status: external_doc]
- [readme] MICCA filters low quality sequences according to the maximum allowed expected error rate: "filters low quality sequences according to the maximum allowed **expected
  error (EE) rate** %"
- [results] the stringent quality filtering approach used by UPARSE, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold: "the stringent quality filtering approach used by UPARSE4, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold"
- [results] Combined with the requirement of a minimum read length, this behaviour causes the loss of a large fraction of the sequence data: "Combined with the requirement of a minimum read length, this behaviour causes the loss of a large fraction of the sequence data"
