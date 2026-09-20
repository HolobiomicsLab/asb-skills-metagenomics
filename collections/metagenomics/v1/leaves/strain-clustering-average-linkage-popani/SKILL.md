---
name: strain-clustering-average-linkage-popani
description: Use when you have inStrain profile objects for multiple samples (e.g.,
  NICU infant metagenomes mapped to a dereplicated genome set) and need strain-level
  identities to test whether specific strains are shared between related individuals.
when_to_use_negative:
- The analysis goal is species- or genome-level comparisons rather than strain-level
  sharing (genome de-replication at 98% ANI with dRep is the appropriate step instead)
- The input is raw reads or .bam files without inStrain profile objects having been
  built first — 'inStrain profile' must be run per sample before 'inStrain compare'
- Samples share no sub-species with sufficient coverage-overlap, so popANI comparisons
  cannot resolve shared strains
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: inStrain
  role: Runs 'inStrain compare' on profile objects to obtain popANI values and clusters
    sub-species into strains via average hierarchical clustering at 99.999% ANI
  repo: github.com/MrOlm/instrain
- name: dRep
  role: Dereplicates the genome set at 98% ANI prior to read mapping, reducing multi-mapped
    reads that would confound strain comparisons
  repo: github.com/MrOlm/drep
- name: SciPy
  role: Performs Fisher's exact test on two-infant strain sharing and rank-sum (Mann-Whitney)
    test comparing MZ vs DZ twin sharing
- name: Pandas
  role: Builds the strain-by-infant presence/absence matrix and computes pairwise
    Jaccard similarity of strain composition
- name: Seaborn
  role: Draws the Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property
    associations)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:d6534c7fdbf96c03cdd28ad70d35516e98a3e02ad74efe3ff8fddc0ad303de8d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - SciPy
  - Seaborn
  - dRep
  - inStrain
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# strain-clustering-average-linkage-popani

## Summary

Clusters inStrain sub-species into individual microbial strains by running 'inStrain compare' on profile objects and applying 'average' hierarchical clustering with a 99.999% popANI threshold, then builds a strain-by-infant presence/absence matrix for strain-sharing statistics.

## When to use

Use when you have inStrain profile objects for multiple samples (e.g., NICU infant metagenomes mapped to a dereplicated genome set) and need strain-level identities to test whether specific strains are shared between related individuals.

## When NOT to use

- The analysis goal is species- or genome-level comparisons rather than strain-level sharing (genome de-replication at 98% ANI with dRep is the appropriate step instead)
- The input is raw reads or .bam files without inStrain profile objects having been built first — 'inStrain profile' must be run per sample before 'inStrain compare'
- Samples share no sub-species with sufficient coverage-overlap, so popANI comparisons cannot resolve shared strains

## Inputs

- inStrain profile objects built per sample from .bam files mapped to the concatenated dereplicated genome FASTA
- Concatenated dereplicated genome set (e.g., dRep output at 98% ANI)
- Infant metadata (sibling/twin relationships, birth-day, birth weight, gestational age)

## Outputs

- Pairwise popANI and coverage-overlap comparison table across all samples ('inStrain compare' output)
- Strain identities per infant (Supplemental Table S4 analog)
- Strain-by-infant presence/absence matrix and pairwise Jaccard similarity values
- Sibling-sharing enrichment p-values (Fisher's exact test) and MZ-vs-DZ rank-sum test results

## How to apply

Map reads from each sample to the concatenated dereplicated genome set (dereplicated at ~98% ANI to reduce multi-mapped reads) and run 'inStrain profile' per sample. Run 'inStrain compare' between all profile-object pairs to obtain pairwise popANI and coverage-overlap values. Cluster sub-species into individual strains using 'average' hierarchical clustering with a threshold of 99.999% ANI, recording strain identities per infant (analogous to Supplemental Table S4). Define strain detection per sample as a sub-species present at >= 0.5 unmaskedBreadth, build the strain-by-sample presence/absence matrix, and compute pairwise Jaccard similarity for downstream tests (e.g., Fisher's exact test on strains present in exactly two infants, Mann-Whitney comparison of MZ vs DZ sharing).

## Related tools

- **inStrain** (Runs 'inStrain compare' on profile objects to obtain popANI values and clusters sub-species into strains via average hierarchical clustering at 99.999% ANI) — github.com/MrOlm/instrain
- **dRep** (Dereplicates the genome set at 98% ANI prior to read mapping, reducing multi-mapped reads that would confound strain comparisons) — github.com/MrOlm/drep
- **SciPy** (Performs Fisher's exact test on two-infant strain sharing and rank-sum (Mann-Whitney) test comparing MZ vs DZ twin sharing)
- **Pandas** (Builds the strain-by-infant presence/absence matrix and computes pairwise Jaccard similarity of strain composition)
- **Seaborn** (Draws the Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property associations))

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Strain identities per infant reproduce the Supplemental Table S4-style inventory: the majority of strains are found in only a single infant
- Clustering uses the exact 'average' hierarchical linkage with 99.999% ANI threshold on popANI values from inStrain compare
- Strains present in two and only two infants show statistically significant enrichment of sibling sharing (Fisher's exact test), while MZ twins share no more strains than DZ twins
- Pairwise Jaccard similarity increases with birth-day proximity and physiological similarity (gestational age, birth weight), matching Figure 3 patterns
- Detected sub-species pass the >= 0.5 unmaskedBreadth detection threshold

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain (Supplemental Figure S3)
- It is difficult to distinguish microbiome diversity evolved in situ from that introduced by immigration, so shared strains may reflect either transmission or parallel acquisition
- Raw comparison values and strain identities are deferred to Supplemental Table S4 and are not included in the main text

## Evidence

- [readme] inStrain compare quick-start invocation: [evidence span withheld — anchor status: external_doc]
