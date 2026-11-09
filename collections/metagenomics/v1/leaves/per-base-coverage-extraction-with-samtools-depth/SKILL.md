---
name: per-base-coverage-extraction-with-samtools-depth
description: Use when you have a BAM from mapping read subsets onto a specific reference
  FASTA and need the per-base covered fraction, gap identification, or coverage progression
  across iterative read-subset expansion cycles.
when_to_use_negative:
- You need assembly-level completeness metrics (N50, contig count) rather than read-level
  per-base coverage of a single reference sequence
- The reference is an entire multi-contig genome where aggregate depth statistics
  are desired instead of a single-target gap-infilling assessment
- The input is not an alignment (no BAM/SAM from mapping the reads to the target contig)
  — depth extraction requires a coordinate-sorted alignment
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: SAMtools
  role: Provides 'samtools depth -a' to extract per-base pileup coverage for every
    position of the cas operon reference, including zero-coverage bases
- name: bowtie2
  role: Maps each cycle's seed read subset and the full raw dataset onto the cas operon
    reference in end-to-end paired-end mode to produce the BAM input for depth extraction
- name: CasCollect
  role: Generates the seed read subsets at each '-cycle N' expansion stop whose coverage
    of the target operon is being quantified
  repo: https://github.com/sandialabs/CasCollect
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:43d1801a1eb36bef09a7776fb107d8ad8c0c09e6cb1eff164f76a010ec250a6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - HMMER
  - SAMtools
  - bowtie2
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# per-base coverage extraction with samtools depth

## Summary

Quantifies sequencing read coverage of a target reference (e.g. the largest ground water cas operon contig) at single-base resolution using 'samtools depth -a', enabling computation of the fraction of reference bases covered by a read subset. This is used to evaluate how CasCollect read-subset expansion infills coverage gaps versus the full raw dataset.

## When to use

Use when you have a BAM from mapping read subsets onto a specific reference FASTA and need the per-base covered fraction, gap identification, or coverage progression across iterative read-subset expansion cycles.

## When NOT to use

- You need assembly-level completeness metrics (N50, contig count) rather than read-level per-base coverage of a single reference sequence
- The reference is an entire multi-contig genome where aggregate depth statistics are desired instead of a single-target gap-infilling assessment
- The input is not an alignment (no BAM/SAM from mapping the reads to the target contig) — depth extraction requires a coordinate-sorted alignment

## Inputs

- BAM file from bowtie2 end-to-end paired-end mapping of a read subset onto a target contig (e.g. the largest ground water cas operon reference FASTA)
- CasCollect seed read subsets generated per '-cycle N' (0 to 5) at 95% vsearch identity match
- Complete raw paired-end FASTQ dataset (baseline mapping, e.g. SRR10598175)

## Outputs

- Per-base depth table covering every reference position (samtools depth -a output)
- Fraction of reference bases covered per read set (percent operon coverage)
- Coverage progression table (e.g. cas_operon_coverage_progression.csv) relating percent coverage to expansion cycle count plus full-dataset baseline

## How to apply

Map the read set of interest (e.g. CasCollect seed subsets generated at each '-cycle N' expansion stop, with default 120 Cas protein HMMs and 95% vsearch identity match) onto the extracted target contig with bowtie2 in end-to-end paired-end mode. Run 'samtools depth -a' on the resulting BAM so that every reference position, including zero-coverage bases, is reported — the '-a' flag is essential, since omitting it silently drops gaps and inflates apparent coverage. From the depth output compute the fraction of operon bases with depth > 0 to obtain percent coverage, and tabulate coverage versus cycle count together with a full-dataset baseline (e.g. into cas_operon_coverage_progression.csv). In the ground water cas operon study, this revealed initial seed coverage of ~33.2%, rising to 96.1% after one expansion cycle with no gaps remaining, and 99.5% after two cycles, indicating saturation.

## Related tools

- **SAMtools** (Provides 'samtools depth -a' to extract per-base pileup coverage for every position of the cas operon reference, including zero-coverage bases)
- **bowtie2** (Maps each cycle's seed read subset and the full raw dataset onto the cas operon reference in end-to-end paired-end mode to produce the BAM input for depth extraction)
- **CasCollect** (Generates the seed read subsets at each '-cycle N' expansion stop whose coverage of the target operon is being quantified) — https://github.com/sandialabs/CasCollect

## Examples

```
samtools depth -a cas_operon_mapped.bam > cas_operon_per_base_depth.txt
```

## Evaluation signals

- samtools depth output includes all reference positions (contiguous position numbering) confirming '-a' was used; zero-depth rows appear where gaps exist
- Reproduced coverage progression matches reported values: ~33.2% for the initial seed set, ~96.1% after one expansion cycle, ~99.5% after two cycles (within a few percent tolerance)
- No gaps remain in the depth table after one expansion cycle for the ground water cas operon
- Coverage saturates by cycle two (cycles 2–5 remain at or near 99.5%), consistent with matched reads being removed from the database between cycles to avoid redundant searches
- Full-dataset baseline coverage is computed and included in the progression table as a comparison point

## Limitations

- Without the '-a' flag, samtools depth omits zero-coverage positions, so coverage gaps would be missed and per-base coverage fractions inflated
- Per-base coverage of a read subset depends on the subset expansion settings (default 95% identity match, five cycles); changing '-cycle' or '-match' changes the reads available to cover the target
- Percent-coverage metrics alone do not report depth magnitude — positions with depth 1 and depth 100 both count as 'covered'

## Evidence

- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command.: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [methods] -match number       percent match for vsearch: "-match number       percent match for vsearch"
- [results] This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
