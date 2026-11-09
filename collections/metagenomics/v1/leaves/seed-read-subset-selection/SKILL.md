---
name: seed-read-subset-selection
description: Use when raw, unassembled Illumina sequencing reads must be reduced to
  a small seed-read subset matching cas proteins or CRISPR repeats before targeted
  assembly, especially for large or metagenomic datasets where full assembly is too
  slow.
when_to_use_negative:
- The input is already an assembled genome or contig set — seed generation targets
  raw, unassembled reads and adds no value there.
- You need a complete de novo assembly of the whole metagenome rather than only CRISPR/Cas
  (or other targeted specialty gene) regions.
- The genes of interest lack any profile HMM or reference nucleotide sequence to search
  against, and no user-defined seed FASTA can be supplied.
edam_operation: http://edamontology.org/operation_0346
tools:
- name: CasCollect
  role: Python wrapper that parses seed-mode flags and combines matching reads from
    all enabled modes into seed_reads.fastq
  repo: github.com/sandialabs/CasCollect
- name: FragGeneScanPlus
  role: Translates each raw sequencing read into its most likely open reading frame
    for protein-mode seed detection
  repo: hallamlab/FragGeneScanPlus
- name: HMMER
  role: Searches FragGeneScanPlus-translated reads against the 120 Cas protein family
    profile HMMs (or user -hmm directory)
- name: VSEARCH
  role: DNA-mode identity matching of reads to the user-defined CRISPR repeat query
    FASTA
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:d5476e2cc31b2274cd825887210cbedbb5e5cce3349e56b2a72b5cec635ba51c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CasCollect
  - FragGeneScanPlus
  - HMMER
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0346
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Seed-read subset selection for targeted assembly (CasCollect seed generation)

## Summary

Generates a compact 'seed' read set from raw Illumina FASTQ reads that match cas genes or CRISPR repeats, using CasCollect's three optional seed-generation modes: protein-mode HMM search of translated reads, DNA-mode VSEARCH identity matching to repeat sequences, and user-defined FASTA seeds. This avoids assembling entire genomes and primes only the CRISPR/Cas-encoding regions for assembly.

## When to use

Use when raw, unassembled Illumina sequencing reads must be reduced to a small seed-read subset matching cas proteins or CRISPR repeats before targeted assembly, especially for large or metagenomic datasets where full assembly is too slow.

## When NOT to use

- The input is already an assembled genome or contig set — seed generation targets raw, unassembled reads and adds no value there.
- You need a complete de novo assembly of the whole metagenome rather than only CRISPR/Cas (or other targeted specialty gene) regions.
- The genes of interest lack any profile HMM or reference nucleotide sequence to search against, and no user-defined seed FASTA can be supplied.

## Inputs

- Raw Illumina short-read FASTQ files (single or paired-end)
- User-defined nucleotide query FASTA of CRISPR/other repeat sequences (for --nucl, -query file.fasta)
- User-defined seed FASTA (for --seed, -define file.fasta)
- Optional user-defined profile HMM directory (for --noprot, -hmm)
- Built-in resources: 120 Cas protein family profile HMMs and 105 CRISPR repeat FASTA files bundled with CasCollect

## Outputs

- seed_reads.fastq — combined seed-read set from all enabled modes, priming the read-subset expansion step

## How to apply

Parse the CasCollect command line to determine which seed-generation modes are enabled (--noprot disables protein mode; --nucl with -query file.fasta enables DNA mode; --seed with -define file.fasta enables user-defined mode, and --nucl requires -query while --noprot pairs with -hmm for user HMM directories). In default protein mode, translate each raw read with FragGeneScanPlus into its most likely open reading frame and search the translations with HMMER against the default 120 Cas protein family profile HMMs (or a user's -hmm directory), keeping matching reads. In DNA mode (--nucl), run VSEARCH identity matching of reads against the user's nucleotide repeat FASTA — CasCollect ships 105 CRISPR repeat sequences as individual FASTA files — and add matching reads to the pool. In user-defined mode (--seed -define file.fasta), accept the user's FASTA entries directly as seeds, bypassing both searches. Combine seeds from all enabled modes into a single seed-read set written as seed_reads.fastq, ready to prime the downstream vsearch read-subset expansion.

## Related tools

- **CasCollect** (Python wrapper that parses seed-mode flags and combines matching reads from all enabled modes into seed_reads.fastq) — github.com/sandialabs/CasCollect
- **FragGeneScanPlus** (Translates each raw sequencing read into its most likely open reading frame for protein-mode seed detection) — hallamlab/FragGeneScanPlus
- **HMMER** (Searches FragGeneScanPlus-translated reads against the 120 Cas protein family profile HMMs (or user -hmm directory))
- **VSEARCH** (DNA-mode identity matching of reads to the user-defined CRISPR repeat query FASTA) — https://github.com/torognes/vsearch

## Examples

```
CasCollect --seed -define user_seeds.fasta reads_1.fastq reads_2.fastq
```

## Evaluation signals

- The module executes without error for each enabled flag combination, honoring the constraints that --nucl requires -query and --noprot pairs with -hmm.
- seed_reads.fastq exists, is a valid FASTQ/FASTA of read sequences, and is non-empty only when the corresponding search returned matches.
- Downstream, both CasCollect and a complete assembly pipeline should recover the same known cas genes, as demonstrated for the E. coli KLY, P. aeruginosa VA-134 and S. pyogenes M39 isolate genomes.
- CasCollect's seed-primed pipeline should complete in far less time than full assembly (e.g. 23.3/17.4 min for E. coli KLY), since only a small read subset is expanded and assembled.
- Seeds from protein, DNA and user-defined modes are merged without duplicates into a single seed_reads.fastq.

## Limitations

- The paper does not benchmark the seed-generation modes in isolation, so acceptance is limited to correct execution and a valid, non-empty seed_reads.fastq when matches are returned.
- Seed sensitivity is bounded by the query resources: the default 120 Cas protein profile HMMs and the 105 bundled CRISPR repeats; novel divergent systems require user-supplied HMMs, repeat queries or seed FASTAs.
- No software version numbers for CasCollect or its third-party dependencies are documented, complicating reproducibility.

## Evidence

- [results] CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes: "we developed CasCollect. CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes"
- [results] HMMER default query set of 120 Cas protein family profile HMMs: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [results] DNA mode uses VSEARCH identity matching against user-defined CRISPR repeat sequences: "DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats"
- [results] 105 CRISPR repeat sequences shipped as individual FASTA files: "We have included 105 CRISPR repeat sequences (13) as individual Fasta files with CasCollect"
- [readme] CasCollect detects Cas genes and CRISPR arrays from unassembled raw Illumina reads: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
- [intro] Seed reads prime targeted assembly of only CRISPR/Cas genomic regions: "identifying reads that match cas genes and/or CRISPR arrays and using these ‘seed’ reads to prime the targeted assembly"
