---
name: scaffold-breadth-metric-computation
description: Use when profiling genome populations from metagenomic read mappings
  and you need per-scaffold coverage, breadth (fraction of reference bases covered
  ≥1 read), and nucleotide diversity from sorted .bam files plus a reference .fasta.
when_to_use_negative:
- You only need fast read coverage/abundance estimates without microdiversity — a
  tool like CoverM's `coverm genome`/`coverm contig` is the article's suggested alternative
  for plain coverage and breadth
- Your .bam contains unpaired reads or pairs lacking proper orientation/insert size,
  minimum mapQ, or minimum nucleotide identity — these pairs are filtered out by default,
  so the metrics will not reflect them
- The task is strain-level comparison of two profiles rather than single-profile scaffold
  metrics — use inStrain's `compare` operation instead
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: inStrain
  role: Provides the profiling logic this skill reconstructs — per-position coverage,
    breadth, microdiversity, and expected-breadth computation
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Used to calculate coverage (number of reads aligned) at each position in the
    provided nucleotide sequence file and to interact with .bam/.sam files
- name: Numpy
  role: Computes per-position nucleotide diversity as 1 minus clonality at positions
    passing the 5-read min_cov threshold
- name: Pandas
  role: Assembles the per-scaffold metrics into the scaffold_info.tsv table
- name: CoverM
  role: Alternative referenced by the article for fast genome coverage/breadth calculation
    when microdiversity profiling is not needed
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:ea7bebfd4b6f96e9aff9471ca3c6191e8b02e78b3b974ff4a0cf800542b2d916
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - CoverM
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scaffold-breadth-metric-computation

## Summary

Reconstructs inStrain's per-scaffold coverage, breadth, and nucleotide-diversity profiling module: tally per-position read depth and A/C/G/T counts from a sorted .bam against a reference .fasta, then aggregate mean/median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity per scaffold. This is why the skill exists — breadth and microdiversity are the metrics inStrain reports for accurate genome comparisons.

## When to use

Use when profiling genome populations from metagenomic read mappings and you need per-scaffold coverage, breadth (fraction of reference bases covered ≥1 read), and nucleotide diversity from sorted .bam files plus a reference .fasta.

## When NOT to use

- You only need fast read coverage/abundance estimates without microdiversity — a tool like CoverM's `coverm genome`/`coverm contig` is the article's suggested alternative for plain coverage and breadth
- Your .bam contains unpaired reads or pairs lacking proper orientation/insert size, minimum mapQ, or minimum nucleotide identity — these pairs are filtered out by default, so the metrics will not reflect them
- The task is strain-level comparison of two profiles rather than single-profile scaffold metrics — use inStrain's `compare` operation instead

## Inputs

- Sorted .bam file of reads mapped to a reference genome
- Reference genome .fasta file
- min_cov threshold (default 5 reads per position)

## Outputs

- scaffold_info.tsv (tab-separated table with scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, nucleotide diversity)

## How to apply

Load the sorted .bam with Samtools together with the reference .fasta, tallying per-position read depth and A/C/G/T base counts from properly paired primary alignments; by default all un-paired reads are removed and filters are applied to read pairs in combination. With Numpy, compute per-position nucleotide diversity as 1 minus clonality (the sum of squared A/C/G/T base frequencies), restricting to positions with coverage above the min_cov threshold of 5 reads. Aggregate per-scaffold statistics — average and median coverage, breadth (fraction of bases covered by ≥1 read), and unmasked breadth excluding masked reference bases — and compute expected breadth from mean coverage using inStrain's model expectedBreadth = -1*e^(-0.883*coverage)+1. Assemble the metrics with Pandas into a tab-separated scaffold_info.tsv listing scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity. A key decision point is the 5-read minimum, which mirrors inStrain's default SNV-calling filter that a position must have at least 5 reads mapping to it.

## Related tools

- **inStrain** (Provides the profiling logic this skill reconstructs — per-position coverage, breadth, microdiversity, and expected-breadth computation) — https://github.com/MrOlm/instrain
- **Samtools** (Used to calculate coverage (number of reads aligned) at each position in the provided nucleotide sequence file and to interact with .bam/.sam files)
- **Numpy** (Computes per-position nucleotide diversity as 1 minus clonality at positions passing the 5-read min_cov threshold)
- **Pandas** (Assembles the per-scaffold metrics into the scaffold_info.tsv table)
- **CoverM** (Alternative referenced by the article for fast genome coverage/breadth calculation when microdiversity profiling is not needed) — https://github.com/wwood/CoverM

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- scaffold_info.tsv contains all metric columns: scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity
- A 20x simulated mapping (e.g., pIRS-simulated reads) yields observed breadth ≈ 1, matching the documented expectation of 100% genome breadth (every base covered by ≥1 read) at 20x coverage
- Nucleotide diversity is computed only at positions with coverage greater than the min_cov threshold of 5 reads
- Expected breadth tracks the inStrain model expectedBreadth = -1*e^(-0.883*coverage)+1 for each scaffold's mean coverage
- Unmasked breadth is lower than or equal to raw breadth when the reference contains masked bases

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Reads from positions with fewer than 5 reads of coverage are excluded from nucleotide-diversity computation, so low-coverage scaffolds receive no diversity estimate
- Only properly paired primary alignments contribute: un-paired reads are removed by default, and pairs must map in proper orientation within the expected insert size and meet user-defined mapQ and nucleotide-identity minima
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads that would otherwise distort per-position base counts

## Evidence
