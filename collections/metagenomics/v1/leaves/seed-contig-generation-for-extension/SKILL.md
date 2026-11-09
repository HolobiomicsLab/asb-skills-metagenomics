---
name: seed-contig-generation-for-extension
description: 'Use when metagenomic reads need de novo-assembled seed contigs before
  a contig-extension step: input is adaptor-trimmed FASTQ and the downstream tool
  expects a FASTA seed contig set.'
when_to_use_negative:
- A seed contig set already exists (e.g. the examples folder's BKV_seed_1000_867.fa)
  — skip assembly and feed that FASTA directly to the extender
- Reads still contain adaptors — preprocess fastq by removing adaptors first, since
  extension quality depends on it
- The goal is reference-based mapping rather than de novo assembly of viral metagenomic
  reads
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: SPAdes
  role: De novo assembler run in metaSPAdes mode (-meta) with k=21,33,55,77 to produce
    the seed contigs
  repo: https://github.com/dengzac/contig-extender
- name: metaSPAdes
  role: Benchmark metagenomic assembler used by the authors to generate the initial
    seed contigs to be extended
- name: ContigExtender
  role: Downstream consumer of the seed contigs; performs recursive extension via
    the r-OLC strategy
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:0b6ff12dd46c5e8e623ad1c45de49da6f251b0ba0959e0c13381284b54ad424a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - SPAdes
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# seed-contig-generation-for-extension

## Summary

Generate the initial seed contigs that ContigExtender consumes by running SPAdes 3.13.0 in metaSPAdes mode (-meta) with multi-k sizes 21, 33, 55, 77 on adaptor-preprocessed metagenomic FASTQ reads, then passing the resulting contigs.fasta downstream for recursive extension.

## When to use

Use when metagenomic reads need de novo-assembled seed contigs before a contig-extension step: input is adaptor-trimmed FASTQ and the downstream tool expects a FASTA seed contig set.

## When NOT to use

- A seed contig set already exists (e.g. the examples folder's BKV_seed_1000_867.fa) — skip assembly and feed that FASTA directly to the extender
- Reads still contain adaptors — preprocess fastq by removing adaptors first, since extension quality depends on it
- The goal is reference-based mapping rather than de novo assembly of viral metagenomic reads

## Inputs

- adaptor-preprocessed metagenomic paired-end FASTQ reads (e.g. simulated BKV dataset such as BKV_250_50_0.01_0_.fastq)
- SPAdes 3.13.0 installed with metaSPAdes mode available

## Outputs

- contigs.fasta containing assembled seed contigs in FASTA format, one per input scaffold, ordered/suitable for ContigExtender input

## How to apply

Load the adaptor-preprocessed metagenomic FASTQ reads (the authors recommend removing adaptors before extension) and run SPAdes 3.13.0 with the '-meta' option to enable MetaSPAdes mode, keeping all other parameters default. Specify the multi-k assembly with k-mer sizes 21, 33, 55, 77 for iterative de Bruijn graph construction, since metaSPAdes is the benchmark assembler the ContigExtender authors used to generate initial seed contigs. Collect the assembled sequences from the SPAdes output directory as contigs.fasta; this file is the seed contig set consumed by ContigExtender's recursive Overlap Layout Candidates (r-OLC) strategy. The paper sets no numeric assembly-quality threshold, so acceptance is that contigs.fasta exists in FASTA format with non-empty contig sequences suitable as seeds for extension.

## Related tools

- **SPAdes** (De novo assembler run in metaSPAdes mode (-meta) with k=21,33,55,77 to produce the seed contigs) — https://github.com/dengzac/contig-extender
- **metaSPAdes** (Benchmark metagenomic assembler used by the authors to generate the initial seed contigs to be extended)
- **ContigExtender** (Downstream consumer of the seed contigs; performs recursive extension via the r-OLC strategy) — https://github.com/dengzac/contig-extender

## Examples

```
spades.py --meta -k 21,33,55,77 -1 BKV_reads_1.fq -2 BKV_reads_2.fq -o bkv_spades_out
```

## Evaluation signals

- contigs.fasta exists in the SPAdes output directory and is valid FASTA
- contig sequences are non-empty and structurally suitable as seed contigs for extension (the paper reports no numeric assembly-quality threshold)
- the FASTQ input was adaptor-preprocessed as recommended by the authors before assembly/extension
- k-mer sizes used in the SPAdes run match 21, 33, 55, 77 and the -meta flag was active
- downstream ContigExtender run accepts the seed file without format errors and reports extendable contigs

## Limitations

- De novo assembly of metagenomic reads yields contigs that are often very fragmented and rarely longer than a few kb, so seeds typically need extension
- The paper reports no numeric assembly-quality threshold, so there is no cutoff-based acceptance check on the seeds
- SPAdes version (3.13.0) and parameter choices are fixed by the benchmark; other assemblies may not reproduce the paper's comparisons

## Evidence

- [intro] Since metaSPAdes has been adopted by many labs, we will use it as our benchmark tool to generate the initial seed contigs to be extended.: "Since metaSPAdes has been adopted by many labs, we will use it as our benchmark tool to generate the initial seed contigs to be extended."
- [abstract] the contigs generated from the de novo assembly are often very fragmented and rarely longer than a few kilo base pairs (kb): "the contigs generated from the de novo assembly are often very fragmented and rarely longer than a few kilo base pairs (kb)"
- [readme] For best results, preprocess fastq by removing adaptors before attemping extension: "For best results, preprocess fastq by removing adaptors before attemping extension"
- [methods] The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig: "The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig"
- [intro] Recently, the popular assembler SPAdes and its variant, metaSPAdes, specially designed for metagenomic applications, have emerged to be the tools of choice: "Recently, the popular assembler SPAdes and its variant, metaSPAdes, specially designed for metagenomic applications, have emerged to be the tools of choice"
