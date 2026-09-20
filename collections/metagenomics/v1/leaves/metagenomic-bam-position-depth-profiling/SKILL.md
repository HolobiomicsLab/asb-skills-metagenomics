---
name: metagenomic-bam-position-depth-profiling
description: Use when you have a sorted .bam mapped to a reference .fasta and need
  per-scaffold coverage, breadth, expected breadth, and nucleotide-diversity metrics
  for genome populations in a metagenome.
when_to_use_negative:
- You only need fast whole-genome coverage/relative abundance without microdiversity
  — CoverM is a faster dedicated coverage calculator
- The task is strain-level comparison between profiles rather than single-sample profiling
  — use inStrain compare on existing profile objects
- The .bam is not sorted or contains unpaired/mis-mapped reads you cannot filter,
  since un-paired reads are removed by default and pairs must meet mapQ and nucleotide-identity
  minimums
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: python package that profiles co-occurring genome populations, computing coverage,
    breadth, nucleotide diversity, and SNVs from the .bam/.fasta pair
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: interacts with .bam and .sam files; used to calculate coverage of each position
    in the provided nucleotide sequence file
- name: Numpy
  role: vectorized computation of per-position nucleotide diversity (1 minus clonality
    as sum of squared base frequencies)
- name: Pandas
  role: assembles per-scaffold metrics into the scaffold_info.tsv table
- name: CoverM
  role: alternative fast tool cited for calculating coverage and breadth when microdiversity
    profiling is not needed
  repo: github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:3768f54f203855830900cb2a2ba4a269cca628e670254ed61db6da21d927d834
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CoverM
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metagenomic-bam-position-depth-profiling

## Summary

Reconstruct inStrain's per-position profiling of a sorted metagenomic .bam against a reference .fasta: tally per-position read depth and A/C/G/T base counts from filtered paired reads, then compute coverage, breadth (including unmasked breadth), expected breadth, and nucleotide diversity per scaffold.

## When to use

Use when you have a sorted .bam mapped to a reference .fasta and need per-scaffold coverage, breadth, expected breadth, and nucleotide-diversity metrics for genome populations in a metagenome.

## When NOT to use

- You only need fast whole-genome coverage/relative abundance without microdiversity — CoverM is a faster dedicated coverage calculator
- The task is strain-level comparison between profiles rather than single-sample profiling — use inStrain compare on existing profile objects
- The .bam is not sorted or contains unpaired/mis-mapped reads you cannot filter, since un-paired reads are removed by default and pairs must meet mapQ and nucleotide-identity minimums

## Inputs

- sorted .bam file of reads mapped to a reference genome
- reference genome .fasta file
- optional filtering parameters (min mapQ, min nucleotide identity, expected insert size)

## Outputs

- scaffold_info.tsv with per-scaffold length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity
- per-position depth and A/C/G/T base-count table underlying diversity calculations

## How to apply

Load the sorted .bam with Samtools together with the reference .fasta and, for each scaffold, tally per-position read depth and A/C/G/T base counts from properly paired primary alignments; inStrain performs careful filtering of reads in the .bam to reduce erroneous mapping (un-paired reads removed by default, pairs mapped in proper orientation within expected insert size, minimum mapQ, minimum nucleotide identity). Compute per-position nucleotide diversity as 1 minus clonality (sum of squared A/C/G/T base frequencies) at positions with coverage above the default min_cov threshold of 5 reads, the same minimum read count required for SNV calling. Aggregate per-scaffold statistics — average and median coverage, breadth (fraction of bases covered by at least one read), and unmasked breadth excluding masked reference bases — and compute expected breadth from mean coverage using the inStrain model expectedBreadth = -1*e^(-0.883*coverage)+1. Assemble the metrics with Pandas into a scaffold_info.tsv listing scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity. Detection of a sub-species in a sample was defined as that genome being present with >= 0.5 unmaskedBreadth, so unmasked breadth doubles as a presence criterion.

## Related tools

- **inStrain** (python package that profiles co-occurring genome populations, computing coverage, breadth, nucleotide diversity, and SNVs from the .bam/.fasta pair) — github.com/MrOlm/instrain
- **Samtools** (interacts with .bam and .sam files; used to calculate coverage of each position in the provided nucleotide sequence file)
- **Numpy** (vectorized computation of per-position nucleotide diversity (1 minus clonality as sum of squared base frequencies))
- **Pandas** (assembles per-scaffold metrics into the scaffold_info.tsv table)
- **CoverM** (alternative fast tool cited for calculating coverage and breadth when microdiversity profiling is not needed) — github.com/wwood/CoverM

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- scaffold_info.tsv contains all expected metric columns (scaffold, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, nucleotide diversity)
- a 20x simulated mapping yields observed breadth ≈ 1, matching the documented expectation of 100% genome breadth at 20x coverage
- expectedBreadth recomputed from mean coverage via -1*e^(-0.883*coverage)+1 matches the reported expected breadth column
- nucleotide diversity is only reported at positions with coverage greater than the min_cov threshold of 5 reads
- breadth minus unmasked breadth equals the fraction of reference bases that are masked

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads that would inflate coverage
- Careful filtering of reads in the .bam is required; erroneous mapping inflates coverage and diversity metrics
- Raw benchmarking values are deferred to Supplemental Table S1 rather than the main text

## Evidence

- [readme] inStrain profile command from README quick start: [evidence span withheld — anchor status: external_doc]
