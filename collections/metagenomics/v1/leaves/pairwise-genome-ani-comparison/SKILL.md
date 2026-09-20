---
name: pairwise-genome-ani-comparison
description: Use when you need sensitive strain-level comparison of microbial populations
  across metagenomic samples, e.g. testing whether related individuals share more
  strains than unrelated ones.
when_to_use_negative:
- You only need coarse coverage and breadth of genomes per sample — CoverM or standard
  mapping-based quantification suffices without popANI comparison.
- You need species-level community profiling from reference databases (e.g. MetaPhlAn-style)
  rather than strain-level resolution on a custom genome set.
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Runs 'profile' on each sample's .bam and 'compare' across profiles to compute
    ConANI/popANI and coverage overlap; also performs hierarchical clustering of sub-species
    into strains
  repo: github.com/MrOlm/instrain
- name: dRep
  role: Dereplicates the genome set at 98% ANI before mapping to reduce multi-mapped
    reads
  repo: github.com/MrOlm/drep
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files during profiling
    and comparison
- name: Bowtie 2
  role: Maps reads from each sample to all representative sub-species genomes concatenated
    together with default settings
- name: SciPy
  role: Fisher's exact test on two-infant strain sharing and Mann-Whitney rank-sum
    test comparing MZ vs DZ twin strain sharing
- name: Pandas
  role: Manipulates the comparison and strain-identity tables when building presence/absence
    and Jaccard matrices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:2085d8fef611dd9cfb179e7b8ccfa9b9e10f807f9f93891b95095b2bde08e7be
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
  - dRep
  - inStrain
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pairwise-genome-ani-comparison

## Summary

Use inStrain compare to obtain all pairwise ConANI and popANI values between metagenomic samples, then cluster sub-species into strains with 'average' hierarchical clustering at a 99.999% ANI threshold to quantify strain sharing across samples.

## When to use

Use when you need sensitive strain-level comparison of microbial populations across metagenomic samples, e.g. testing whether related individuals share more strains than unrelated ones.

## When NOT to use

- You only need coarse coverage and breadth of genomes per sample — CoverM or standard mapping-based quantification suffices without popANI comparison.
- You need species-level community profiling from reference databases (e.g. MetaPhlAn-style) rather than strain-level resolution on a custom genome set.

## Inputs

- Sorted .bam files of reads mapped to a concatenated set of dereplicated representative sub-species genomes
- inStrain profile objects (one per sample) produced by 'inStrain profile'
- Reference genome FASTA used for mapping

## Outputs

- All pairwise popANI and ConANI comparison values between samples
- Coverage-overlap values for each sample pair
- Strain identities per sample (analogous to Supplemental Table S4)
- Strain-by-sample presence/absence matrix and pairwise Jaccard similarity values

## How to apply

Map each sample's reads to a concatenated dereplicated genome set with Bowtie 2 (dereplicate at 98% ANI, e.g. with dRep, to reduce multi-mapped reads) and run 'inStrain profile' on each resulting .bam. Then run 'inStrain compare' across the inStrain profile objects to obtain all pairwise ConANI and popANI values plus coverage-overlap metrics. Cluster sub-species into individual strains using 'average' hierarchical clustering with a threshold of 99.999% ANI, recording strain identities per sample (Supplemental Table S4). Define a sub-species as detected in a sample when present at >= 0.5 unmaskedBreadth, build a strain-by-sample presence/absence matrix, and compute pairwise Jaccard similarity of strain composition for downstream statistical tests (Fisher's exact test for sibling enrichment, rank-sum for MZ vs DZ, correlation with birth-day distance, birth weight, gestational age).

## Related tools

- **inStrain** (Runs 'profile' on each sample's .bam and 'compare' across profiles to compute ConANI/popANI and coverage overlap; also performs hierarchical clustering of sub-species into strains) — github.com/MrOlm/instrain
- **dRep** (Dereplicates the genome set at 98% ANI before mapping to reduce multi-mapped reads) — github.com/MrOlm/drep
- **Samtools** (Required by inStrain for interacting with .bam and .sam files during profiling and comparison)
- **Bowtie 2** (Maps reads from each sample to all representative sub-species genomes concatenated together with default settings)
- **SciPy** (Fisher's exact test on two-infant strain sharing and Mann-Whitney rank-sum test comparing MZ vs DZ twin strain sharing)
- **Pandas** (Manipulates the comparison and strain-identity tables when building presence/absence and Jaccard matrices)

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2 -o compare_out
```

## Evaluation signals

- Pairwise comparisons output contains ConANI and popANI values for every sample pair with associated coverage-overlap values
- Strain clustering used 'average' hierarchical clustering with the 99.999% ANI threshold, matching the published workflow
- Sub-species detection calls use the >= 0.5 unmaskedBreadth cutoff
- The majority of identified strains should be found in only a single sample; two-sample strains should show statistically significant sharing enrichment among siblings
- Jaccard similarity of strain composition increases with birth-day proximity and physiological similarity (birth weight, gestational age), while MZ twins do not exceed DZ twins in sharing

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads; skipping it inflates spurious similarity
- It is difficult to distinguish microbiome diversity that evolved in situ from that introduced by immigration

## Evidence

- [readme] $ inStrain compare inStrain_profile1 inStrain_profile2: [evidence span withheld — anchor status: external_doc]
