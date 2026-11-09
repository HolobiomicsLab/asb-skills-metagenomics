---
name: fastq-read-subset-management
description: Use when you have a small FASTA/FASTQ seed read set (e.g. cas-gene or
  CRISPR-repeat matches) and a large raw FASTQ dataset, and you need to recruit neighboring
  reads to prime targeted assembly without assembling the whole dataset.
when_to_use_negative:
- The full genome/metagenome is small enough that complete assembly and annotation
  is acceptable — CasCollect's targeted expansion exists to avoid complete assembly
- Input is already an assembly (contigs/FASTA assemblies) rather than raw unassembled
  reads; the skill operates on FASTQ read datasets
- Protein-level or local-alignment recruitment is required — VSEARCH in this pipeline
  performs nucleotide identity matching only
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: VSEARCH
  role: Performs the per-cycle identity search of all reads against the current seed/subset
    set at the 95% (-match) threshold
  repo: https://github.com/torognes/vsearch
- name: CasCollect
  role: Python wrapper orchestrating the match-and-remove expansion cycles and writing
    expanded_reads.fastq
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: 'Upstream seed generation: identifies cas-gene-matching reads that seed the
    expansion'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:e90c002b7b15952f558e2e2f8a3a4a408268e5cddd9536028f4f91269eb9f430
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - CasCollect
  - HMMER
  - VSEARCH
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fastq-read-subset-management

## Summary

Iteratively expands a set of seed FASTQ reads into a targeted subset by cycling VSEARCH identity matching at 95%, removing matched reads from the search database each cycle so no read is searched twice. This is the read subset expansion stage that primes CasCollect's targeted assembly.

## When to use

Use when you have a small FASTA/FASTQ seed read set (e.g. cas-gene or CRISPR-repeat matches) and a large raw FASTQ dataset, and you need to recruit neighboring reads to prime targeted assembly without assembling the whole dataset.

## When NOT to use

- The full genome/metagenome is small enough that complete assembly and annotation is acceptable — CasCollect's targeted expansion exists to avoid complete assembly
- Input is already an assembly (contigs/FASTA assemblies) rather than raw unassembled reads; the skill operates on FASTQ read datasets
- Protein-level or local-alignment recruitment is required — VSEARCH in this pipeline performs nucleotide identity matching only

## Inputs

- Seed read set: user-defined FASTA of seed reads or seed reads derived from HMM/query matching (e.g. cas-gene HMMER hits or CRISPR repeat matches)
- Target short-read sequencing data in FASTQ format (optionally gzipped FASTQ, readable by VSEARCH)
- Configuration parameters: -match identity threshold (default 95%) and -cycle number of expansion cycles (default five)

## Outputs

- expanded_reads.fastq — the final expanded read subset used as SPAdes/metaSPAdes assembly input
- Per-cycle read recruitment counts reported to the user

## How to apply

Load the seed read set (user-defined FASTA of seed reads or HMM/query-derived seed reads from the seed generation step) together with the target FASTQ read file(s). Run VSEARCH to search all remaining reads against the current seed/subset set using a 95% identity threshold (the -match default), then add the newly matched reads to the expanded subset and remove them from the search database so they are not re-searched in later rounds — this eliminates redundant searches. Repeat the match-and-remove cycle for the configured number of cycles (-cycle, default five), and write the final expanded subset to expanded_reads.fastq with per-cycle recruitment counts reported. Because matched reads are removed from the database between cycles, each cycle only searches the still-unmatched portion of the dataset.

## Related tools

- **VSEARCH** (Performs the per-cycle identity search of all reads against the current seed/subset set at the 95% (-match) threshold) — https://github.com/torognes/vsearch
- **CasCollect** (Python wrapper orchestrating the match-and-remove expansion cycles and writing expanded_reads.fastq) — https://github.com/sandialabs/CasCollect
- **HMMER** (Upstream seed generation: identifies cas-gene-matching reads that seed the expansion)

## Examples

```
python CasCollect.py -i reads.fastq -seed seeds.fasta -match 95 -cycle 5
```

## Evaluation signals

- expanded_reads.fastq exists, is valid FASTQ, and is non-empty
- The expanded subset grows monotonically across cycles — matched reads are only added, never lost
- Default run settings are 95% identity (-match) over five cycles (-cycle), matching the published defaults
- Per-cycle recruitment counts are reported and search cost shrinks each cycle because matched reads are removed from the database
- Downstream SPAdes/metaSPAdes assembly uses only the seed plus expanded subset, and cas genes are still identified (as validated on E. coli KLY, P. aeruginosa VA-134, and S. pyogenes M39 reads)

## Limitations

- Identity threshold is fixed per run (default 95%); divergent CRISPR/Cas neighborhoods beyond 95% identity will not be recruited without relaxing -match
- A bounded number of cycles (default five) may leave distant flanking regions unrecruited in large insert regions
- Only matched reads are removed from the database between cycles; reads below threshold are re-searched every cycle
- Pipeline depends on external tools (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) being present in the user's path; CasCollect checks and reports missing software

## Evidence

- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command.: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps: "The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps"
- [readme] Before running, **CasCollect** will check for the necessary software in the users path and denote any missing software.: "Before running, **CasCollect** will check for the necessary software in the users path and denote any missing software."
