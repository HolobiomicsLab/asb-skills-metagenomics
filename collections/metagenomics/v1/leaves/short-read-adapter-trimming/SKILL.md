---
name: short-read-adapter-trimming
description: Use when raw FASTQ reads downloaded from SRA (e.g. accession SRR961514)
  are to be assembled with a de Bruijn graph assembler such as MetaCortex and adapter
  contamination must be removed first.
when_to_use_negative:
- Reads have already been adapter-trimmed or were supplied pre-trimmed by the sequencing
  provider
- The downstream tool consumes SRA or BAM files directly and handles adapter removal
  itself
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: trim-galore
  role: Adapter-trimming software used to trim reads before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
- name: SRA toolkit
  role: Provides the fastq-dump command used to obtain the raw reads prior to trimming
  repo: https://github.com/ncbi/sra-tools/wiki
- name: MetaCortex
  role: Downstream metagenome assembler consuming the trimmed reads in its two-stage
    CTX-based workflow
  repo: https://github.com/SR-Martin/metacortex
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:9e167d5f24cb8e8e766b325a9950283c2cb9484ab59ba6911b1a7bc0c0a6d1e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelixKrueger/TrimGalore
  tools:
  - Conda
  - GCC
  - MetaCortex
  - MetaQUAST
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# short-read-adapter-trimming

## Summary

Adapter-trim raw next-generation sequencing reads with trim-galore prior to metagenomic de Bruijn graph assembly, as done in the MetaCortex Mock A/Mock B viral community benchmarks.

## When to use

Use when raw FASTQ reads downloaded from SRA (e.g. accession SRR961514) are to be assembled with a de Bruijn graph assembler such as MetaCortex and adapter contamination must be removed first.

## When NOT to use

- Reads have already been adapter-trimmed or were supplied pre-trimmed by the sequencing provider
- The downstream tool consumes SRA or BAM files directly and handles adapter removal itself

## Inputs

- Raw paired-end Illumina FASTQ reads downloaded from SRA (e.g. SRR961514 via fastq-dump)

## Outputs

- Adapter-trimmed FASTQ read files ready for de Bruijn graph assembly
- trim-galore trimming report files

## How to apply

After retrieving reads with the SRA toolkit's fastq-dump command (e.g. for the Mock B community, accession SRR961514), trim the reads using trim-galore adapter-trimming software before assembly. Trimming is a prerequisite step in the MetaCortex workflow: the trimmed FASTQ files are the input that is subsequently imported into the de Bruijn graph to produce CTX binary files. Perform the trimming before any assembly stage (MetaCortex two-stage run, or baseline assemblers like MEGAHIT, metaSPAdes, Ray Meta) so that all assemblers receive identical adapter-free input. Use the same trimmed reads for both the MetaCortex assembly and comparator assemblers so benchmark metrics such as genome fraction, misassemblies, and mismatches/indels per 100 kb from MetaQUAST are directly comparable.

## Related tools

- **trim-galore** (Adapter-trimming software used to trim reads before assembly) — https://github.com/FelixKrueger/TrimGalore
- **SRA toolkit** (Provides the fastq-dump command used to obtain the raw reads prior to trimming) — https://github.com/ncbi/sra-tools/wiki
- **MetaCortex** (Downstream metagenome assembler consuming the trimmed reads in its two-stage CTX-based workflow) — https://github.com/SR-Martin/metacortex

## Examples

```
fastq-dump SRR961514 && trim_galore SRR961514.fastq
```

## Evaluation signals

- trim-galore produces a per-input-file trimming report summarising adapter removal and quality trimming statistics
- Trimmed FASTQ output files exist and contain no adapter-read-through sequences detectable in a spot-check
- The same trimmed reads feed both MetaCortex and the comparator assemblers (MEGAHIT, Ray Meta, metaSPAdes) so benchmark inputs are identical
- Downstream MetaQUAST metrics on Mock A/Mock B (e.g. genome fraction 99.89%/99.71%) are consistent with the reported benchmark values when the trimmed input is used

## Limitations

- LLVM will not compile MetaCortex; Mac users must install GCC and set CC to the GCC binary, though this affects the assembly stage rather than trimming itself
- The tutorial workflow prescribes trimming before assembly; skipping it may alter de Bruijn graph construction and the reported genome fraction and misassembly metrics

## Evidence

- [intro] Trim reads with adapter-trimming software before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [intro] Trimming precedes the two-stage MetaCortex assembly: "Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
- [intro] Raw reads are obtained from SRA before trimming: "Download the reads using the fastq-dump command from the SRA toolkit::"
- [intro] SRR961514 is the tutorial dataset to be trimmed: "	fastq-dump SRR961514"
- [readme] Trim Galore tool identity and purpose from README: [evidence span withheld — anchor status: external_doc]
