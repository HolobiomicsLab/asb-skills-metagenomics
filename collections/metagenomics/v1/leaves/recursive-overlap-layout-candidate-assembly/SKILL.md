---
name: recursive-overlap-layout-candidate-assembly
description: Use when de novo metagenomic assembly (e.g., MetaSPAdes) yields seed
  contigs of only a few kb that need extension using raw sequencing reads, and contig
  accuracy must be preserved (e.g., >98% BLAST alignment to a reference viral genome).
when_to_use_negative:
- Input contigs are already complete genomes or longer than the target stop-length,
  so extension would only trigger circular-genome repeat termination
- Reads have not had adaptors removed — the README warns extension results will be
  suboptimal without this preprocessing
- The goal is general de novo assembly of reads rather than extension of existing
  seed contigs; use a DBG assembler like MetaSPAdes, MEGAHIT, or IDBA-UD instead
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0781
tools:
- name: ContigExtender
  role: Core tool performing recursive Overlap Layout Candidates (r-OLC) contig extension
    via Docker wrapper ./dist/extender_wrapper
  repo: https://github.com/dengzac/contig-extender
- name: PRINSEQ
  role: DUST complexity filtering of reads before extension, controlled by --complex-threshold
    (default 15)
- name: Bowtie2
  role: Computes read-to-contig alignments used by r-OLC; --enable-pair-constraint
    enforces paired-end orientation and insert size constraints (maxins 500)
- name: metaSPAdes
  role: Generates the initial seed contigs that are subsequently extended
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:d9fee862466444faebf9a433a7fc58c6d4b555acfbb899632662453f00c91c43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - Bowtie2
  - ContigExtender
  - PRINSEQ
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# recursive-overlap-layout-candidate-assembly

## Summary

Extends fragmented MetaSPAdes seed contigs from viral metagenomics data using ContigExtender's recursive Overlap Layout Candidates (r-OLC) strategy, which iteratively appends overlapping reads while exploring alternative branches to produce longer, highly accurate contigs.

## When to use

Use when de novo metagenomic assembly (e.g., MetaSPAdes) yields seed contigs of only a few kb that need extension using raw sequencing reads, and contig accuracy must be preserved (e.g., >98% BLAST alignment to a reference viral genome).

## When NOT to use

- Input contigs are already complete genomes or longer than the target stop-length, so extension would only trigger circular-genome repeat termination
- Reads have not had adaptors removed — the README warns extension results will be suboptimal without this preprocessing
- The goal is general de novo assembly of reads rather than extension of existing seed contigs; use a DBG assembler like MetaSPAdes, MEGAHIT, or IDBA-UD instead

## Inputs

- FASTA file of de novo assembled seed contigs (e.g., 58 MetaSPAdes seed contigs from NIBSC sample 26)
- FASTQ file of unpaired sequencing reads (or paired --m1/--m2 FASTQ files)
- Optional reference viral genome FASTA (e.g., BKV.fasta) for accuracy verification

## Outputs

- output/contigs.fasta containing extended contigs sorted by length and named in input order (e.g., >reference_1 reference)
- Per-contig length gains (final contigs typically 1.7-10 kb; largest extension ~5.8 kb)
- BLAST alignment results showing percent single-segment alignment to reference viral genomes

## How to apply

Preprocess the FASTQ by removing adaptors, then invoke the ContigExtender Docker wrapper (./dist/extender_wrapper) on the FASTA of seed contigs and the reads. Keep PRINSEQ's DUST complexity filtering active (complex-threshold 15) and run r-OLC with defaults: min-overlap-length 15, extend-tolerance 2.5, min-branch-score 5, branch-limit 1, stop-length 250, and 8 threads for alignments. For paired-end data, pass --m1/--m2 with --enable-pair-constraint so alignments with incorrect orientation or fragments longer than 500 bp are excluded. Collect extended contigs from output/contigs.fasta (sorted by length, named by input order), select contigs extended by >=200 bp, and validate each by nucleotide BLAST against its reference viral genome, expecting >98% single-segment alignment. The r-OLC strategy recursively branches on alternative extending paths when a candidate meets min-branch-score, and stop-length guards against infinite extension of circular genomes by terminating when a substring of that size repeats.

## Related tools

- **ContigExtender** (Core tool performing recursive Overlap Layout Candidates (r-OLC) contig extension via Docker wrapper ./dist/extender_wrapper) — https://github.com/dengzac/contig-extender
- **PRINSEQ** (DUST complexity filtering of reads before extension, controlled by --complex-threshold (default 15))
- **Bowtie2** (Computes read-to-contig alignments used by r-OLC; --enable-pair-constraint enforces paired-end orientation and insert size constraints (maxins 500))
- **metaSPAdes** (Generates the initial seed contigs that are subsequently extended)

## Examples

```
./dist/extender_wrapper --coverage 50 examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq
```

## Evaluation signals

- Expected extension yield: 26 of 58 NIBSC sample 26 seed contigs extended; select contigs with >=200 bp gain
- Accuracy check: nucleotide BLAST of each extended contig against its reference viral genome shows >98% single-segment alignment for at least 24 of 26 final contigs
- Output format invariant: output/contigs.fasta is sorted by length and contigs named in input order (e.g., >reference_1 reference)
- Extension terminates properly: stop-length 250 prevents infinite extension of circular genomes via repeated substrings
- Final contig lengths fall in the expected 1.7-10 kb range with largest extension of 5.8 kb

## Limitations

- Higher --extend-tolerance values (3.0-3.5) can force extension but risk higher chances of false extension
- Output contigs for a single input may include multiple alternatives; branch-limit (default 1) caps how many alternative contigs are considered
- Contigs generated by de novo assembly are often fragmented and rarely longer than a few kb, so only extendable contigs gain length; not all seed contigs will extend
- Only Linux is supported, and Docker is required to run the wrapper

## Evidence

- [readme] ContigExtender employs a novel recursive Overlap Layout Candidates (r-OLC) strategy that explores multiple extending paths: "ContigExtender employs a novel recursive Overlap Layout Candidates (r-OLC) strategy that explores multiple extending paths to achieve longer and highly accurate contigs."
- [readme] This parameter is passed to PRINSEQ's DUST complexity filter, is run on the reads: "[0-100] This parameter is passed to PRINSEQ's DUST complexity filter, is run on the reads."
- [readme] Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded.: "Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded."
- [intro] contig accuracy, which is measured by similarity to NCBI viral reference nucleotide sequences using nucleotide BLAST: "contig accuracy, which is measured by similarity to NCBI viral reference nucleotide sequences using nucleotide BLAST"
- [discussion] For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb: "For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb"
