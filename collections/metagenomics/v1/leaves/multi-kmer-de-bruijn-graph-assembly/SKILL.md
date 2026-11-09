---
name: multi-kmer-de-bruijn-graph-assembly
description: Use when adaptor-preprocessed metagenomic FASTQ reads must be assembled
  de novo to produce fragmented seed contigs that feed a contig-extension step such
  as ContigExtender.
when_to_use_negative:
- Input is already a set of assembled contigs — this skill only produces the initial
  seed contigs before extension.
- Reads still contain adaptors — the ContigExtender authors require adaptor removal
  before extension and downstream quality suffers if preprocessing is skipped.
- The goal is isolates/single-genome assembly rather than metagenomic data — metaSPAdes
  mode is designed for metagenomic applications.
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: SPAdes
  role: de novo assembler engine (v3.13.0) run in metaSPAdes mode to generate seed
    contigs
- name: metaSPAdes
  role: metagenomic variant of SPAdes used as the benchmark tool to generate the initial
    seed contigs to be extended
- name: ContigExtender
  role: downstream consumer of the seed contigs; extends them via recursive Overlap
    Layout Candidates (r-OLC) strategy
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:688ebfc470cef8b0302e86573d1d30db2a0abbe857f1e8193d397d81907c5004
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - SPAdes
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# multi-kmer-de-bruijn-graph-assembly

## Summary

Generates seed contigs for downstream contig extension by running de novo metagenomic assembly with metaSPAdes (SPAdes 3.13.0, -meta mode) using a multi-k de Bruijn graph (k=21,33,55,77), all other parameters at default.

## When to use

Use when adaptor-preprocessed metagenomic FASTQ reads must be assembled de novo to produce fragmented seed contigs that feed a contig-extension step such as ContigExtender.

## When NOT to use

- Input is already a set of assembled contigs — this skill only produces the initial seed contigs before extension.
- Reads still contain adaptors — the ContigExtender authors require adaptor removal before extension and downstream quality suffers if preprocessing is skipped.
- The goal is isolates/single-genome assembly rather than metagenomic data — metaSPAdes mode is designed for metagenomic applications.

## Inputs

- adaptor-preprocessed metagenomic FASTQ reads (e.g., simulated BKV dataset BKV_250_50_0.01_0_.fastq from the examples folder)

## Outputs

- contigs.fasta — FASTA file of assembled seed contigs (output of SPAdes) to be extended by ContigExtender

## How to apply

Load adaptor-preprocessed metagenomic FASTQ reads (single- or paired-end) and run SPAdes 3.13.0 in metaSPAdes mode via the '-meta' flag, explicitly setting multi-k sizes 21,33,55,77 for iterative de Bruijn graph construction; leave all other parameters at default. metaSPAdes is chosen because it is the metagenomic-specific variant of SPAdes and serves as the benchmark generator of seed contigs in the ContigExtender workflow. Collect the assembled contigs from the SPAdes output directory as a FASTA file (contigs.fasta) to serve as the seed contig set consumed by ContigExtender. No numeric assembly-quality threshold is applied downstream, so validation is that contigs.fasta exists, is valid FASTA, and contains non-empty contig sequences.

## Related tools

- **SPAdes** (de novo assembler engine (v3.13.0) run in metaSPAdes mode to generate seed contigs)
- **metaSPAdes** (metagenomic variant of SPAdes used as the benchmark tool to generate the initial seed contigs to be extended)
- **ContigExtender** (downstream consumer of the seed contigs; extends them via recursive Overlap Layout Candidates (r-OLC) strategy) — https://github.com/dengzac/contig-extender

## Examples

```
spades.py --meta -k 21,33,55,77 -1 reads_1.fq -2 reads_2.fq -o metaspades_out
```

## Evaluation signals

- contigs.fasta exists in the SPAdes output directory and is valid FASTA format
- contigs.fasta contains non-empty assembled contig sequences suitable as seed contigs for extension
- the exact k-mer sizes 21,33,55,77 were used (multi-k iterative de Bruijn graph construction)
- reads were adaptor-preprocessed before assembly, as recommended by the authors
- seed contigs are short/fragmented as expected (a few kb), consistent with the known limitation of de novo assembly that ContigExtender is designed to address

## Limitations

- No numeric assembly-quality threshold is defined; acceptance is purely structural (contigs.fasta exists with non-empty contigs).
- De novo assembled contigs are often very fragmented and rarely longer than a few kilobases — extension (e.g., ContigExtender) is required for longer sequences.
- SPAdes version is pinned at 3.13.0 with default parameters apart from -meta and k-mer sizes; other versions/settings are not validated by the paper.

## Evidence

- [intro] Since metaSPAdes has been adopted by many labs, we will use it as our benchmark tool to generate the initial seed contigs to be extended.: "Since metaSPAdes has been adopted by many labs, we will use it as our benchmark tool to generate the initial seed contigs to be extended."
- [abstract] the contigs generated from the de novo assembly are often very fragmented and rarely longer than a few kilo base pairs (kb).: "the contigs generated from the de novo assembly are often very fragmented and rarely longer than a few kilo base pairs (kb)."
- [readme] The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig.: "The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig."
- [intro] Recently, the popular assembler SPAdes and its variant, metaSPAdes, specially designed for metagenomic applications, have emerged to be the tools of choice: "Recently, the popular assembler SPAdes and its variant, metaSPAdes, specially designed for metagenomic applications, have emerged to be the tools of choice"
