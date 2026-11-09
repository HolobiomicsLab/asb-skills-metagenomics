---
name: metagenomic-contig-extension
description: Use when metaSPAdes or another de novo assembler has produced viral metagenomic
  seed contigs of only a few kb and the goal is to lengthen them with paired or unpaired
  FASTQ reads while checking accuracy against NCBI viral reference sequences.
when_to_use_negative:
- The reads are reference-based and a de novo extension strategy is unnecessary because
  viral references already exist for direct assembly
- The seed contigs are already near-complete (e.g. ~11.9 kb full viral genomes); extension
  offers no gain and only adds false-extension risk
- Runtime is critical and read complexity screening cannot be afforded — note this
  still requires disabling DUST via --complex-threshold -1, accepting the loss of
  complexity filtering
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: ContigExtender
  role: 'Primary tool: Dockerized r-OLC contig extension of seed contigs with overlapping
    reads'
  repo: https://github.com/dengzac/contig-extender
- name: metaSPAdes
  role: Generates the initial seed contigs to be extended; also serves as the end-to-end
    assembly benchmark
- name: Bowtie2
  role: Performs read alignments with --enable-pair-constraint enforcing orientation
    and insert size (--maxins, default 500)
- name: PRINSEQ
  role: DUST complexity filtering of reads at complex-threshold 15 (disabled with
    -1)
- name: PRICE
  role: Baseline contig-extension tool for length comparison on the same seeds and
    reads (Table 2)
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:e5ccc22a216a58d56aad13141c5e9722fa3b8abdc0b961d1fc4ef959e54eeb32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - Bowtie2
  - ContigExtender
  - PRICE
  - PRINSEQ
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# metagenomic-contig-extension

## Summary

Automated recursive Overlap Layout Candidates (r-OLC) extension of fragmented de novo metagenomic assembly contigs using ContigExtender, which iteratively incorporates overlapping reads to gain several kb of length while maintaining contig accuracy validated by nucleotide BLAST.

## When to use

Use when metaSPAdes or another de novo assembler has produced viral metagenomic seed contigs of only a few kb and the goal is to lengthen them with paired or unpaired FASTQ reads while checking accuracy against NCBI viral reference sequences.

## When NOT to use

- The reads are reference-based and a de novo extension strategy is unnecessary because viral references already exist for direct assembly
- The seed contigs are already near-complete (e.g. ~11.9 kb full viral genomes); extension offers no gain and only adds false-extension risk
- Runtime is critical and read complexity screening cannot be afforded — note this still requires disabling DUST via --complex-threshold -1, accepting the loss of complexity filtering

## Inputs

- FASTA file of de novo assembled seed contigs (e.g. MetaSPAdes output)
- FASTQ file of unpaired reads, or two FASTQ files of paired mates via --m1/--m2
- Optional reference genome FASTA (e.g. BKV.fasta) for accuracy verification
- Optional --coverage estimate of sequencing depth (default 10)

## Outputs

- output/contigs.fasta with extended contigs sorted by length and named reference_N in input order
- Per-contig gained length (extended length minus seed length), tabulated for contigs extended by >0.2 kb
- BLAST alignment of each extended contig to its NCBI viral reference for accuracy assignment

## How to apply

Preprocess FASTQ reads by removing adaptors, then install Docker, clone dengzac/contig-extender, and run ./dist/extender_wrapper with the seed-contig FASTA as positional reference and the reads FASTQ (or --m1/--m2 for paired mates with --enable-pair-constraint and --maxins 500 so Bowtie2 excludes misoriented or >500 bp fragments). Run the r-OLC extension with defaults (min-overlap-length 15, extend-tolerance 2.5, coverage 10, min-branch-score 5, branch-limit 1, stop-length 250, threads 8) plus PRINSEQ DUST complexity filtering at complex-threshold 15, or set --complex-threshold -1 for faster execution at the risk of no complexity screening. For forced extension, raise --extend-tolerance to [2.5, 3.5], which increases false-extension risk. Collect extended contigs from output/contigs.fasta (sorted by length, named reference_N in input order), compute gained length (extended minus seed length), and validate each contig by nucleotide BLAST against the corresponding NCBI viral reference; correct runs yield >98% single-segment alignment for most contigs and gains of roughly 2 kb (e.g. seed average 2.8 kb to final 4.8 kb across 49 human/animal contigs, with 45 of 49 extensions high quality and non-chimeric).

## Related tools

- **ContigExtender** (Primary tool: Dockerized r-OLC contig extension of seed contigs with overlapping reads) — https://github.com/dengzac/contig-extender
- **metaSPAdes** (Generates the initial seed contigs to be extended; also serves as the end-to-end assembly benchmark)
- **Bowtie2** (Performs read alignments with --enable-pair-constraint enforcing orientation and insert size (--maxins, default 500))
- **PRINSEQ** (DUST complexity filtering of reads at complex-threshold 15 (disabled with -1))
- **PRICE** (Baseline contig-extension tool for length comparison on the same seeds and reads (Table 2))

## Examples

```
./dist/extender_wrapper examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq --coverage 50
```

## Evaluation signals

- Gained length per contig (extended minus seed length) is >0.2 kb for the tabulated contigs; reported gains average ~2 kb (2.8 kb seed to 4.8 kb final)
- Nucleotide BLAST of extended contigs against the NCBI viral reference shows >98% single-segment alignment for the large majority (24 of 26 NIBSC final contigs)
- No chimeric extensions are counted: 45 of 49 combined human/animal extensions are high quality and non-chimeric
- Output contigs in output/contigs.fasta are sorted by length and named reference_N in the order they appeared in the input
- On the simulated BASV benchmark, reproduced ContigExtender lengths reach ~6.7 kb (100 bp/10x/error 0.01) and ~11.9 kb (250 bp/20x/error 0.01) from 1 kb seeds

## Limitations

- Only Linux is supported, and the tool runs exclusively through a Docker image downloaded by ./dist/extender_wrapper
- Raising --extend-tolerance above the 2.5 default (e.g. 3.0) forces extension but risks higher chances of false extension
- Disabling DUST complexity filtering (--complex-threshold -1) speeds execution but removes read-complexity screening; --stop-length 250 exists specifically to prevent circular genomes from infinite extension
- Longest reported extension is ~5.8 kb and final NIBSC contigs reach 1.7-10 kb; seeds rarely extend beyond several kb of gain, and supplementary comparison tables (S1/S2) were not reproducible from the provided document

## Evidence

- [readme] ContigExtender employs a novel recursive Overlap Layout Candidates (r-OLC) strategy that explores multiple extending paths: "ContigExtender employs a novel recursive Overlap Layout Candidates (r-OLC) strategy that explores multiple extending paths"
- [readme] Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded: "Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded."
- [readme] For best results, preprocess fastq by removing adaptors before attemping extension: "For best results, preprocess fastq by removing adaptors before attemping extension"
- [readme] This parameter is passed to PRINSEQ's DUST complexity filter, is run on the reads: "This parameter is passed to PRINSEQ's DUST complexity filter, is run on the reads"
- [discussion] For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb: "For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb"
- [abstract] the contigs generated from the de novo assembly are often very fragmented and rarely longer than a few kilo base pairs (kb): "the contigs generated from the de novo assembly are often very fragmented and rarely longer than a few kilo base pairs (kb)"
