---
name: expected-breadth-coverage-modeling
description: Use when profiling metagenomic read mappings against a reference .fasta
  and you need to distinguish low breadth caused by low coverage from breadth loss
  caused by strain absence or contamination.
when_to_use_negative:
- Input is raw reads or an unsorted .bam that has not been mapped to a reference .fasta
  — breadth modeling requires per-position depth on a reference
- You only need raw coverage/abundance numbers without breadth-vs-coverage interpretation;
  a fast tool like CoverM's 'coverm genome' suffices
- Comparing genomes across samples at strain level (use inStrain compare's ConANI/popANI
  instead of single-profile breadth metrics)
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Computes coverage, breadth, unmasked breadth, and expected breadth per scaffold
    from .bam mappings
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Used by inStrain to read .bam/.sam files and calculate coverage of each position
    in the nucleotide sequence file
- name: Numpy
  role: Computes per-position nucleotide diversity (1 minus clonality) used alongside
    breadth metrics
- name: Pandas
  role: Assembles per-scaffold statistics into scaffold_info.tsv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:a37e23f6173b161b19a0faeb833d27608f2b2d9d58cdc67567c643505ed8ccf5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Expected breadth coverage modeling

## Summary

Models the expected fraction of a reference genome covered by at least one read as a monotonic function of mean mapping depth, using inStrain's expectedBreadth = -1*e^(-0.883*coverage)+1, so that observed scaffold breadth can be judged against coverage-implied expectations.

## When to use

Use when profiling metagenomic read mappings against a reference .fasta and you need to distinguish low breadth caused by low coverage from breadth loss caused by strain absence or contamination.

## When NOT to use

- Input is raw reads or an unsorted .bam that has not been mapped to a reference .fasta — breadth modeling requires per-position depth on a reference
- You only need raw coverage/abundance numbers without breadth-vs-coverage interpretation; a fast tool like CoverM's 'coverm genome' suffices
- Comparing genomes across samples at strain level (use inStrain compare's ConANI/popANI instead of single-profile breadth metrics)

## Inputs

- Sorted .bam file of metagenomic reads mapped to a reference genome
- Reference genome .fasta file
- Per-position read depth and A/C/G/T base counts from properly paired primary alignments
- Minimum coverage threshold (min_cov = 5 reads by default)

## Outputs

- scaffold_info.tsv with columns: scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, nucleotide diversity
- Per-scaffold expected breadth computed as -1*e^(-0.883*coverage)+1

## How to apply

First compute per-position read depth and A/C/G/T base counts from properly paired primary alignments in the sorted .bam using Samtools, then aggregate per-scaffold average and median coverage, breadth (fraction of bases covered by >=1 read), and unmasked breadth excluding masked reference bases. For each scaffold, plug its mean coverage into the inStrain model expectedBreadth = -1*e^(-0.883*coverage)+1 to obtain the breadth expected under that depth if the population were uniformly present. Compare observed breadth and unmasked breadth to expected breadth: observed values far below expected indicate that only part of the genome population is present, while inStrain defines detection of a sub-species as >= 0.5 unmaskedBreadth. Assemble scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity into a scaffold_info.tsv via Pandas for downstream comparisons.

## Related tools

- **inStrain** (Computes coverage, breadth, unmasked breadth, and expected breadth per scaffold from .bam mappings) — https://github.com/MrOlm/instrain
- **Samtools** (Used by inStrain to read .bam/.sam files and calculate coverage of each position in the nucleotide sequence file)
- **Numpy** (Computes per-position nucleotide diversity (1 minus clonality) used alongside breadth metrics)
- **Pandas** (Assembles per-scaffold statistics into scaffold_info.tsv)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- scaffold_info.tsv contains all expected metric columns (scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, nucleotide diversity)
- A simulated 20x mapping yields observed breadth ≈ 1, matching the documented expectation of 100% genome breadth at 20x coverage
- Expected breadth increases monotonically with scaffold mean coverage and is bounded in (0, 1); e.g. 20x coverage gives expected breadth > 0.99
- Observed unmasked breadth >= 0.5 is required for sub-species detection per inStrain's definition
- Unmasked breadth differs from raw breadth only at masked reference bases, so unmaskedBreadth >= breadth must hold

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain, which can depress observed breadth at scaffold edges
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads, which otherwise inflate coverage estimates
- The skill only reports expected breadth from mean coverage; it cannot distinguish in situ evolved diversity from immigrant diversity within covered regions

## Evidence

- [readme] inStrain profile command on bam and fasta inputs: [evidence span withheld — anchor status: external_doc]
