---
name: sra-read-retrieval
description: Use when a benchmark or tutorial pipeline requires the raw reads for
  a specific SRA accession (e.g. SRR961514 Mock B viral community) as FASTQ input
  for downstream trimming and metagenomic assembly.
when_to_use_negative:
- Reads are already available locally as FASTQ and do not need re-download from SRA/ENA
- The target data is a reference genome or annotation rather than archived sequencing
  reads
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SRA toolkit
  role: Provides the fastq-dump command used to download the reads (SRR961514) prior
    to trimming and assembly
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Consumes the downloaded FASTQ reads to trim adapters before MetaCortex assembly
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:a66da759941d62ec6555c185b8841b7f1e722982a7fac154f446102d88a8623c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCC
  - MetaCortex
  - MetaQUAST
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sra-read-retrieval

## Summary

Retrieve raw sequencing reads used in the MetaCortex assembly tutorials (e.g. the Mock B viral community run SRR961514) from NCBI SRA/ENA using the SRA toolkit's fastq-dump command, as the first step before trimming and de Bruijn graph assembly.

## When to use

Use when a benchmark or tutorial pipeline requires the raw reads for a specific SRA accession (e.g. SRR961514 Mock B viral community) as FASTQ input for downstream trimming and metagenomic assembly.

## When NOT to use

- Reads are already available locally as FASTQ and do not need re-download from SRA/ENA
- The target data is a reference genome or annotation rather than archived sequencing reads

## Inputs

- SRA accession identifier (e.g. SRR961514 for the Mock B viral community)
- ENA accession for auxiliary read sets (e.g. ERR2099157 human gut reads)

## Outputs

- FASTQ read files for the requested accession, ready for trim-galore adapter trimming

## How to apply

Install the NCBI SRA toolkit, then run `fastq-dump SRR961514` to download the Mock B viral community reads used by both the MetaQUAST-evaluated Mock B benchmark (MetaCortex k=63 vs MEGAHIT, Ray Meta) and the in silico HIV lab-mix assembly task. The downloaded FASTQ is passed to trim-galore for adapter trimming before MetaCortex assembly. For the HIV mix task, a companion human gut read set (ERR2099157) is fetched from ENA and HIV-1 strain reads (89.6, HXB2, JR-CSF, NL4-3, YU-2) are mixed in at ~2.8% total abundance. Verify success by checking that FASTQ files for both mates are produced and non-empty before proceeding to the assembly stages.

## Related tools

- **SRA toolkit** (Provides the fastq-dump command used to download the reads (SRR961514) prior to trimming and assembly) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Consumes the downloaded FASTQ reads to trim adapters before MetaCortex assembly) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
fastq-dump SRR961514
```

## Evaluation signals

- fastq-dump completes without error and produces FASTQ files for the requested accession (SRR961514)
- The downloaded reads trim cleanly with trim-galore and can be imported by MetaCortex into CTX files
- Assembly of the retrieved reads reproduces the benchmark genome fractions (99.71% for Mock B; 82.563% for the HIV lab mix) when evaluated with MetaQUAST
- For the HIV mix, the retrieved human gut reads (ERR2099157) accept the in-silico-spiked HIV-1 strains at ~2.8% total abundance

## Limitations

- The SRA toolkit must be installed before the tutorial reads can be obtained; the accessions cited are specific to this study's benchmarks
- Large metagenomic read sets may require substantial time and bandwidth to download via fastq-dump

## Evidence

- [intro] SRA toolkit requirement: "The `SRA toolkit <https://github.com/ncbi/sra-tools/wiki>`_ will need to be installed to obtain the reads used in this tutorial"
- [intro] fastq-dump command: "Download the reads using the fastq-dump command from the SRA toolkit::

	fastq-dump SRR961514"
- [intro] trimming before assembly: "and we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
