---
name: snv-calling-from-metagenomic-read-mappings
description: Use when sorted .bam read mappings to a reference .fasta are available
  and the goal is sensitive SNP detection with per-gene localization and synonymous/nonsynonymous
  classification for microdiversity or strain-sharing analyses.
when_to_use_negative:
- The analysis goal is only rapid whole-genome coverage/breadth estimation without
  SNP resolution — inStrain's own docs point to CoverM for that use case
- The mapping contains only un-paired reads, which inStrain removes by default, leaving
  no usable read pairs
- Reference genomes have not been dereplicated; multi-mapped reads inflate calls —
  de-replicate at ~98% ANI first (e.g., with dRep)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Primary tool for read filtering, SNV calling with gene localization and synonymous/nonsynonymous
    identification, and microdiversity profiling
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Interacting with .bam and .sam files and calculating per-position coverage
- name: Prodigal
  role: Annotating open reading frames on the reference to enable per-gene SNP typing
- name: Pandas
  role: Extracting and classifying biallelic SNV positions across samples
- name: SciPy
  role: Statistical support for dN/dS and diversity calculations
- name: Numpy
  role: Numerical computation for per-gene dN/dS
- name: Matplotlib
  role: Plotting per-gene dN/dS and nonsynonymous change distributions
- name: Seaborn
  role: Statistical plotting of within- vs between-infant variant patterns
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:29b7c31b1a995974370834b887b59b9edd48998f0d14bd8779aff72718a9d8cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - Prodigal
  - Samtools
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# snv-calling-from-metagenomic-read-mappings

## Summary

Call SNVs from metagenomic read mappings to reference genomes using inStrain, with read-pair filtering and null-model SNP thresholds, to profile population microdiversity within and between samples (e.g., intra- vs inter-infant variants on a phage genome).

## When to use

Use when sorted .bam read mappings to a reference .fasta are available and the goal is sensitive SNP detection with per-gene localization and synonymous/nonsynonymous classification for microdiversity or strain-sharing analyses.

## When NOT to use

- The analysis goal is only rapid whole-genome coverage/breadth estimation without SNP resolution — inStrain's own docs point to CoverM for that use case
- The mapping contains only un-paired reads, which inStrain removes by default, leaving no usable read pairs
- Reference genomes have not been dereplicated; multi-mapped reads inflate calls — de-replicate at ~98% ANI first (e.g., with dRep)

## Inputs

- Sorted .bam read-mapping files (one per sample) against a reference genome .fasta
- Reference genome nucleotide .fasta (e.g., phage 482_10.ph)
- Prodigal gene calls (.fna) for gene-level profiling

## Outputs

- inStrain profile directory with per-position SNVs, coverage, and nucleotide diversity
- Gene-level coverage, microdiversity, and SNP type table (inStrain profile_genes report)
- Classified biallelic SNV table (e.g., intra- vs inter-infant) and per-gene dN/dS values

## How to apply

Run 'inStrain profile' on each sample's sorted .bam against the reference .fasta; un-paired reads are removed by default and paired reads must map in proper orientation within the expected insert size and pass user-defined minimum mapQ and nucleotide identity (default read-pair ANI 0.95). SNVs require at least 5 reads mapping to the position, more than 5% of reads carrying the variant base, and variant reads exceeding a null model of sequencing error at a false discovery rate of 1e-6. Run 'inStrain profile_genes' with Prodigal gene calls (run in metagenome mode) to obtain per-gene coverage, microdiversity, and SNP type, enabling dN/dS as [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)]. Extract biallelic SNV positions across samples with Pandas and classify each variant (e.g., intra- vs inter-infant) before plotting nonsynonymous change distributions with Matplotlib/Seaborn. Use 'inStrain compare' on the resulting profiles for strain-level comparisons (ConANI and popANI) when shared-strain detection is the goal.

## Related tools

- **inStrain** (Primary tool for read filtering, SNV calling with gene localization and synonymous/nonsynonymous identification, and microdiversity profiling) — https://github.com/MrOlm/instrain
- **Samtools** (Interacting with .bam and .sam files and calculating per-position coverage)
- **Prodigal** (Annotating open reading frames on the reference to enable per-gene SNP typing)
- **Pandas** (Extracting and classifying biallelic SNV positions across samples)
- **SciPy** (Statistical support for dN/dS and diversity calculations)
- **Numpy** (Numerical computation for per-gene dN/dS)
- **Matplotlib** (Plotting per-gene dN/dS and nonsynonymous change distributions)
- **Seaborn** (Statistical plotting of within- vs between-infant variant patterns)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- Validation: the output reproduces the reported pattern that nonsynonymous changes in the tail fiber gene are purged within infants yet selected for between infants
- Gene-level coverage and diversity values match the inStrain profile_genes report
- Every called SNV satisfies the defaults: >= 5 covering reads, > 5% variant-base frequency, and FDR 1e-6 against the sequencing-error null model
- Filtered read pairs map in proper orientation within the expected insert size with read-pair ANI >= 0.95
- Comparison of two profiles via 'inStrain compare' yields ConANI/popANI values consistent with known strain-sharing relationships

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads; otherwise SNVs can be spurious
- It is difficult to distinguish microbiome diversity that is evolved in situ from that introduced by immigration
- inStrain requires Samtools for interacting with .bam and .sam files and Prodigal for annotating open reading frames; missing dependencies block the pipeline
- The null model can be adjusted to account for technologies with different sequencing error rates — defaults may not suit all platforms

## Evidence
