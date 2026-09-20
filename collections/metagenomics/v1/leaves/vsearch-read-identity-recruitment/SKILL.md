---
name: vsearch-read-identity-recruitment
description: Use when raw FASTQ/Illumina reads contain a small set of seed reads of
  interest (e.g. HMM-derived cas-gene hits or CRISPR array repeats) and neighboring
  reads must be recruited to enable targeted assembly without assembling the full
  dataset.
when_to_use_negative:
- Input is already an assembled contig set or a feature/annotation table — this skill
  operates on raw unassembled FASTQ reads only
- The target gene set has no usable seed (no HMM hits, no reference FASTA repeats/genes)
  — the expansion stage requires an initial seed read set to prime recruitment
- Full de novo assembly of the complete metagenome/genome is required rather than
  targeted recovery of a specific operon or locus
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: CasCollect
  role: Python wrapper pipeline that orchestrates the iterative read subset expansion
    cycles, manages matched/unmatched read partitioning, and writes expanded_reads.fastq
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: Performs the percent-identity search of all dataset reads against the current
    seed/subset set in each expansion cycle
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:b268b244be1dad3c8a710085ac3b5cab527e4de282355ab60c3a29c1f75f169c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# vsearch-read-identity-recruitment

## Summary

Iterative VSEARCH-based identity matching that expands a small set of seed reads (e.g. cas-gene or CRISPR-array hits from CasCollect) into a larger read subset from raw FASTQ data, recruiting neighboring reads at a 95% identity threshold over repeated cycles.

## When to use

Use when raw FASTQ/Illumina reads contain a small set of seed reads of interest (e.g. HMM-derived cas-gene hits or CRISPR array repeats) and neighboring reads must be recruited to enable targeted assembly without assembling the full dataset.

## When NOT to use

- Input is already an assembled contig set or a feature/annotation table — this skill operates on raw unassembled FASTQ reads only
- The target gene set has no usable seed (no HMM hits, no reference FASTA repeats/genes) — the expansion stage requires an initial seed read set to prime recruitment
- Full de novo assembly of the complete metagenome/genome is required rather than targeted recovery of a specific operon or locus

## Inputs

- Seed read set as FASTA (user-defined seed reads or HMM/query-derived seed reads from the CasCollect seed-generation step)
- Target raw Illumina sequencing reads in FASTQ format (single or multiple files)
- Optional parameters: -match percent identity threshold (default 95%) and -cycle number of expansion cycles (default 5)

## Outputs

- expanded_reads.fastq — the final expanded read subset in FASTQ format, ready for SPAdes/metaSPAdes assembly
- Per-cycle read recruitment counts reported for each expansion cycle

## How to apply

Load the seed read set (a user-defined FASTA of seed reads or HMM/query-derived seed reads) and the target FASTQ read file(s), then run VSEARCH to search all reads against the current seed/subset set using the percent-match identity threshold (-match, default 95%). Add newly matched reads to the expanded subset and remove them from the search database, since 'This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle'. Repeat the search for the configured number of expansion cycles (-cycle, default five), each cycle using the growing subset as the query against the shrinking unmatched database. Write the final expanded read subset to expanded_reads.fastq and report per-cycle recruitment counts. The 95% identity default was chosen as 'a moderately high value' to balance sensitivity against false recruitment when priming targeted assembly of only CRISPR/Cas genomic regions.

## Related tools

- **CasCollect** (Python wrapper pipeline that orchestrates the iterative read subset expansion cycles, manages matched/unmatched read partitioning, and writes expanded_reads.fastq) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (Performs the percent-identity search of all dataset reads against the current seed/subset set in each expansion cycle) — https://github.com/torognes/vsearch

## Examples

```
./vsearch --usearch_global queries.fsa --db database.fsa --id 0.9 --alnout alnout.txt
```

## Evaluation signals

- expanded_reads.fastq exists, is non-empty, and is in valid FASTQ format
- Recruitment counts grow monotonically across expansion cycles (each cycle's query includes prior matches)
- Default run uses 95% identity (-match) over five cycles (-cycle) as specified; confirm reported parameters match
- Matched reads are removed from the search database between cycles, so per-cycle search time/size shrinks rather than repeating all-vs-all searches
- The expanded subset, when passed to SPAdes/metaSPAdes, yields contigs containing the targeted CRISPR/Cas region comparable to complete-assembly results

## Limitations

- The 95% identity default is 'moderately high' — divergent homologs or distant relatives may fail recruitment and require lowering -match, at the risk of false positives
- A fixed number of expansion cycles (default 5) may under-recruit reads flanking long operons if the locus exceeds the reach of five cycles; -cycle must be increased manually
- Only recruits reads sharing near-identity to existing seed/subset reads; novel loci with no seed representation cannot be discovered at this stage

## Evidence

- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command.: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [readme] CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads.: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
