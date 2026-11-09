---
name: assembly-contiguity-statistics
description: Use when you have contigs from a MetaCortex (or similar de Bruijn graph)
  metagenomic assembly and need to quantify assembly quality as contig counts, N50
  and maximum contig length after a length cutoff.
when_to_use_negative:
- The input is not contigs but raw reads or a reads-to-contig alignment (this skill
  evaluates the assembly output, not the trimming or mapping stages).
- You need per-bin quality metrics such as completeness and strain heterogeneity —
  those require CheckM assessment of MetaBAT2 bins, not assembly statistics.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: MetaCortex
  role: Metagenomic assembler whose contigs output the statistics are computed on;
    runs in two stages producing CTX files then contigs and sequence graphs
  repo: https://github.com/SR-Martin/metacortex.git
- name: trim-galore
  role: Trims adapters and low-quality bases from reads before assembly so the contig
    statistics reflect a quality-filtered input
  repo: https://github.com/FelixKrueger/TrimGalore
- name: SRA toolkit
  role: Provides fastq-dump to obtain the WGS reads used in the assembly
  repo: https://github.com/ncbi/sra-tools/wiki
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:846b0c7217ecf82c8df257ba7908431e4d055f90af4f02480d4d66fade90cf01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - Bowtie2
  - CheckM
  - Conda
  - GCC
  - MetaBAT2
  - MetaCortex
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# assembly-contiguity-statistics

## Summary

Compute metagenomic assembly contiguity statistics (number of contigs ≥500 bp, N50, maximum contig length) from MetaCortex SW assembly output, as demonstrated on a human gut whole-genome shotgun sample to produce comparable assembly summary tables.

## When to use

Use when you have contigs from a MetaCortex (or similar de Bruijn graph) metagenomic assembly and need to quantify assembly quality as contig counts, N50 and maximum contig length after a length cutoff.

## When NOT to use

- The input is not contigs but raw reads or a reads-to-contig alignment (this skill evaluates the assembly output, not the trimming or mapping stages).
- You need per-bin quality metrics such as completeness and strain heterogeneity — those require CheckM assessment of MetaBAT2 bins, not assembly statistics.

## Inputs

- Contigs FASTA output from MetaCortex SW assembly (k=63, min coverage 2, delta 0.8)
- Trimmed reads (trim-galore output FASTQ) used for the assembly

## Outputs

- Filtered contig set (contigs >=500 bp only)
- Assembly summary table: contig count (e.g. 123720), N50 (e.g. 5399), maximum contig length (e.g. 879838 bp)

## How to apply

Assemble trimmed reads with MetaCortex SW using k=63, minimum coverage 2 and delta 0.8; the assembler runs in two stages — parallel import of reads files into CTX de Bruijn graph files, then combining the separate CTX files to output contigs and sequence graphs. Discard contigs shorter than 500 bp from the assembly output before computing statistics. From the filtered FASTA, count the number of retained contigs, compute the N50 (contig length at which 50% of the assembly's bases are contained in contigs of that length or longer) and record the maximum contig length. For the human gut benchmark, correct execution yields 123720 contigs ≥500 bp with N50 5399 and a maximum contig length of 879838 bp; report these alongside downstream MetaBAT2 binning/CheckM quality tables (155 bins, 23 bins >50% complete) as the compiled summary.

## Related tools

- **MetaCortex** (Metagenomic assembler whose contigs output the statistics are computed on; runs in two stages producing CTX files then contigs and sequence graphs) — https://github.com/SR-Martin/metacortex.git
- **trim-galore** (Trims adapters and low-quality bases from reads before assembly so the contig statistics reflect a quality-filtered input) — https://github.com/FelixKrueger/TrimGalore
- **SRA toolkit** (Provides fastq-dump to obtain the WGS reads used in the assembly) — https://github.com/ncbi/sra-tools/wiki

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- Contig count after the ≥500 bp filter matches the target of 123720 contigs for the human gut sample
- N50 of the filtered assembly equals 5399 bp
- Maximum contig length equals 879838 bp
- All reported contig lengths are ≥500 bp (no contigs below the cutoff remain in the summary)
- Assembly summary is consistent with the bin-quality table compiled downstream (155 bins, 23 bins >50% complete, 11 bins with >50% strain heterogeneity)

## Limitations

- LLVM will not compile MetaCortex, so Mac users must install GCC and set CC to point to the GCC binary; building requires 'make MAXK=63 metacortex' for the chosen maximum k-mer size.
- Statistics are tied to specific assembly parameters (k=63, min coverage 2, SW delta 0.8) and the 500 bp contig filter; changing parameters or cutoffs changes the reported contig count and N50.
- The article's worked example downloads reads SRR961514 via fastq-dump; verify accession identity before reproducing benchmark numbers.

## Evidence

- [results] Assembly output filter and statistics target: [evidence span withheld — anchor status: not_found]
- [intro] Two-stage assembly process: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
- [methods] Build with chosen max k-mer: "make MAXK=63 metacortex"
- [intro] Trimming before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [intro] Compile with GCC requirement: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
