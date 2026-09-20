---
name: crispr-cas-targeted-read-capture
description: Use when raw unassembled FASTQ reads must be recruited around known seed
  reads (HMM-derived or user-provided FASTA seeds) to capture a complete CRISPR/Cas
  operon for targeted assembly without assembling the full dataset.
when_to_use_negative:
- The goal is whole-genome or whole-metagenome assembly rather than targeted recovery
  of a specific gene set — CasCollect is designed to avoid complete assembly of the
  raw read dataset
- 'No seed material is available: the expansion stage requires a seed set from cas-gene
  HMM matches or a user-defined FASTA of seed/CRISPR-array repeat reads'
- The target gene family falls outside what the supplied HMM profiles or user reference
  sequences can seed — the pipeline is only reconfigurable for user-provided HMMs
  and/or reference nucleotide sequences
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: Performs the per-cycle identity matching of all dataset reads against the
    current seed/subset set at the 95% percent-match threshold
  repo: https://github.com/torognes/vsearch
- name: CasCollect
  role: Python wrapper that orchestrates the expansion cycles, database pruning between
    cycles, and output of expanded_reads.fastq
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: Generates cas-gene-matching seed reads that prime the expansion stage
- name: SPAdes
  role: Assembles the expanded read subset into targeted contigs (metaSPAdes for metagenomes)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:55ed768c528d60b2a3bce4de50fa28b97f0d0e3bdb78451b7bd507bb0b6bf229
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - CasCollect
  - HMMER
  - SPAdes
  - VSEARCH
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# crispr-cas-targeted-read-capture

## Summary

Iteratively expands an initial set of 'seed' reads (cas-gene or CRISPR-array matches) into a larger read subset from raw Illumina FASTQ data using cycles of VSEARCH identity matching, priming targeted assembly of only CRISPR/Cas loci instead of the whole genome.

## When to use

Use when raw unassembled FASTQ reads must be recruited around known seed reads (HMM-derived or user-provided FASTA seeds) to capture a complete CRISPR/Cas operon for targeted assembly without assembling the full dataset.

## When NOT to use

- The goal is whole-genome or whole-metagenome assembly rather than targeted recovery of a specific gene set — CasCollect is designed to avoid complete assembly of the raw read dataset
- No seed material is available: the expansion stage requires a seed set from cas-gene HMM matches or a user-defined FASTA of seed/CRISPR-array repeat reads
- The target gene family falls outside what the supplied HMM profiles or user reference sequences can seed — the pipeline is only reconfigurable for user-provided HMMs and/or reference nucleotide sequences

## Inputs

- Seed read set: user-defined FASTA of seed reads or HMM/query-derived seed reads
- Raw Illumina short-read FASTQ file(s) from the target isolate or metagenome
- Optional user parameters: -match identity threshold (default 95) and -cycle cycle count (default 5)

## Outputs

- expanded_reads.fastq — the final expanded read subset for downstream SPAdes assembly
- Per-cycle read recruitment counts reporting subset growth

## How to apply

Load the seed read set (a user-defined FASTA of seed reads or reads derived from HMMER hits against the 120 Cas protein family profile HMMs) together with the target FASTQ read file(s). Run VSEARCH in each expansion cycle to search all dataset reads against the current subset at the 95% percent-match identity threshold (default, adjustable with -match). Add newly matched reads to the expanded subset and remove them from the search database, eliminating redundant searches in subsequent cycles. Repeat for the configured number of cycles (-cycle, default five), then write the final recruited reads to expanded_reads.fastq, which feeds the SPAdes/metaSPAdes targeted assembly step. The rationale is that each cycle pulls in reads overlapping the growing subset, walking outward from the seed into the flanking CRISPR/Cas regions.

## Related tools

- **VSEARCH** (Performs the per-cycle identity matching of all dataset reads against the current seed/subset set at the 95% percent-match threshold) — https://github.com/torognes/vsearch
- **CasCollect** (Python wrapper that orchestrates the expansion cycles, database pruning between cycles, and output of expanded_reads.fastq) — https://github.com/sandialabs/CasCollect
- **HMMER** (Generates cas-gene-matching seed reads that prime the expansion stage)
- **SPAdes** (Assembles the expanded read subset into targeted contigs (metaSPAdes for metagenomes))

## Examples

```
vsearch --usearch_global reads.fq --db seed_subset.fsa --id 0.95 --alnout cycle_hits.txt
```

## Evaluation signals

- expanded_reads.fastq exists, is valid FASTQ, and is non-empty
- The read count of the expanded subset grows monotonically across the five default expansion cycles
- The default run used the specified 95% percent-match identity (-match) and five cycles (-cycle)
- Matched reads are removed from the database between cycles — re-searched read overlap should be zero, per the redundant-search elimination design
- Downstream CRISPRCasFinder recovers the expected CRISPR arrays and cas operons from contigs assembled from the expanded subset

## Limitations

- The 95% identity default may miss more divergent flanking reads; the threshold must be tuned via -match for distant homologs
- Five default cycles may under-recruit long flanking regions or over-recruit with too many cycles; -cycle must be adjusted per dataset
- CasCollect depends on external tools (HMMER, VSEARCH, SPAdes, CRISPRCasFinder, BBTools, Seqtk, FragGeneScan) being installed in the user's path; the pipeline checks and flags missing software

## Evidence

- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command.: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps: "The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps"
- [readme] This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"
