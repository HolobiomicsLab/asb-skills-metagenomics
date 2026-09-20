---
name: metagenomic-read-pair-filtering
description: Use when metagenomic reads have been mapped to representative sub-species
  genomes (e.g. concatenated .fasta) and per-scaffold or per-genome coverage, clonality,
  nucleotide diversity, or SNV calls must be computed without contamination from falsely
  mapped read pairs.
when_to_use_negative:
- Input is single-end or fragmented reads where proper-pair orientation and insert-size
  checks cannot be applied — inStrain removes un-paired reads by default, so most
  data would be discarded
- You only need rough coverage/abundance estimates and speed matters more than mapping
  accuracy — a fast tool like CoverM computes coverage without inStrain's pair-level
  filtering
- You are comparing assembled genomes directly (e.g. ANI dereplication with dRep)
  rather than profiling read mappings
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Applies the read-pair filters and computes coverage, clonality, and SNV-based
    nucleotide diversity on the filtered pairs
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Used by inStrain for interacting with .bam and .sam files and calculating
    per-position coverage
  repo: https://github.com/samtools/samtools
- name: Bowtie 2
  role: Maps reads to the concatenated representative sub-species genomes with default
    settings to produce the .bam input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:5a675ab8f3cdef350db0836bf436dcdc3f27123a56834b2e1cd44cfdeabf3f72
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bowtie 2
  - Matplotlib
  - Numpy
  - Pandas
  - Samtools
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metagenomic-read-pair-filtering

## Summary

inStrain's careful .bam read-pair filtering reduces erroneous mappings before coverage, clonality, and SNV-based nucleotide diversity are computed, which is essential for reliable population microdiversity estimates from metagenomic read mappings.

## When to use

Use when metagenomic reads have been mapped to representative sub-species genomes (e.g. concatenated .fasta) and per-scaffold or per-genome coverage, clonality, nucleotide diversity, or SNV calls must be computed without contamination from falsely mapped read pairs.

## When NOT to use

- Input is single-end or fragmented reads where proper-pair orientation and insert-size checks cannot be applied — inStrain removes un-paired reads by default, so most data would be discarded
- You only need rough coverage/abundance estimates and speed matters more than mapping accuracy — a fast tool like CoverM computes coverage without inStrain's pair-level filtering
- You are comparing assembled genomes directly (e.g. ANI dereplication with dRep) rather than profiling read mappings

## Inputs

- Sorted .bam file of read pairs mapped against concatenated representative sub-species .fasta (e.g. via Bowtie 2 with default settings)
- Reference nucleotide sequence file (.fasta) of representative sub-species genomes
- Optional filter parameters: minimum mapQ, minimum nucleotide identity (-l 0.95 default), min_cov 5, min_freq 0.05, SNP FDR 1e-6

## Outputs

- inStrain profile output folder with per-scaffold coverage, clonality, breadth, and SNV-based nucleotide diversity tables
- Filtered set of read pairs used for downstream genome_wide aggregation

## How to apply

Run `inStrain profile <coReads.bam> <reference.fasta>` on each mapping; inStrain automatically applies its read filters to pairs of reads in combination rather than single reads. Filters remove all un-paired reads by default, require pairs to map in proper orientation within an expected insert size, require a user-defined minimum mapQ score, and require pairs to exceed a minimum nucleotide identity (default -l 0.95). For SNV calling, only positions with at least 5 mapping reads, more than 5% of reads carrying the variant base, and variant evidence exceeding the null model at FDR 1e-6 are retained. The rationale is that metagenomic mappings to concatenated reference genomes attract spurious hits from related strains, so filtering in pairs (not singly) lowers the probability of reads being erroneously mapped before diversity metrics are aggregated genome-wide.

## Related tools

- **inStrain** (Applies the read-pair filters and computes coverage, clonality, and SNV-based nucleotide diversity on the filtered pairs) — github.com/MrOlm/instrain
- **Samtools** (Used by inStrain for interacting with .bam and .sam files and calculating per-position coverage) — https://github.com/samtools/samtools
- **Bowtie 2** (Maps reads to the concatenated representative sub-species genomes with default settings to produce the .bam input)

## Examples

```
inStrain profile coReads.bam representative_subspecies.fasta -o inStrain_profile1 -l 0.95 --min_cov 5 --min_freq 0.05 --snp_fdr 1e-6
```

## Evaluation signals

- The inStrain profile output reports filter statistics; the vast majority of retained mappings should be proper pairs passing the mapQ and -l 0.95 nucleotide identity filters
- Detected sub-species should be defined at >= 0.5 unmaskedBreadth, so a correctly filtered run yields genome-level breadth values consistent with true presence/absence
- Downstream inStrain genome_wide aggregation should reproduce Figure 4-style results: Klebsiella with higher nucleotide diversity in C-section-born than vaginally born infants
- SNV calls should respect the built-in thresholds (>= 5 reads, >5% variant frequency, FDR 1e-6); variant sites near the ends of genome fragments may be missed and are a known artifact

## Limitations

- Variant sites near the ends of genome fragments can be missed by inStrain
- Un-paired reads are removed by default, so single-end datasets lose most or all data under this filtering scheme
- Filters can be tuned (minimum reads per position, variant frequency, FDR, null model) to account for technologies with different sequencing error rates; defaults may not suit every platform

## Evidence

- [readme] $ inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1: [evidence span withheld — anchor status: external_doc]
