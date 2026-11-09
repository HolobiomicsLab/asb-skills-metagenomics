---
name: twin-strain-sharing-enrichment-testing
description: Use when metagenomic samples from related and unrelated individuals (e.g.
  NICU infant stool) have inStrain compare popANI profiles and you need to test sibling
  enrichment and MZ-vs-DZ strain-sharing patterns.
when_to_use_negative:
- Strains cannot be resolved because samples were compared with species-level abundance
  tools (e.g. MetaPhlAn) rather than inStrain popANI comparisons
- The question is within-sample microdiversity or nucleotide diversity (e.g. Klebsiella
  diversity by birth mode), not between-sample strain sharing
- Samples lack relationship metadata (sibling/MZ/DZ status, birth day, birth weight,
  gestational age), making the enrichment and correlation tests underpowered
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: inStrain
  role: Profiles each infant sample and runs 'inStrain compare' to obtain pairwise
    popANI and coverage-overlap values used to cluster strains
  repo: github.com/MrOlm/instrain
- name: dRep
  role: De-replicates the genome set (~98% ANI) into representative genomes for the
    concatenated reference used in mapping
  repo: github.com/MrOlm/drep
- name: SciPy
  role: Fisher's exact test for sibling enrichment and rank-sum (Mann-Whitney) test
    for MZ vs DZ sharing
- name: Pandas
  role: Builds the strain-by-infant presence/absence matrix and Jaccard similarity
    table
- name: Numpy
  role: Numerical handling of similarity matrices during statistical testing
- name: Matplotlib
  role: Draws the Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological
    associations)
- name: Seaborn
  role: Statistical plotting of similarity-vs-property associations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:917a9b8dd4d32da4beede6f07b7282dd15af01bed4c7304a527b1705389cf94d
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# twin-strain-sharing-enrichment-testing

## Summary

Tests whether microbial strains (clustered from inStrain popANI profiles at 99.999% ANI) are shared between sibling premature infants more than expected by chance, and whether MZ twins share more strains than DZ twins, reproducing the Figure 3 strain-sharing analysis.

## When to use

Use when metagenomic samples from related and unrelated individuals (e.g. NICU infant stool) have inStrain compare popANI profiles and you need to test sibling enrichment and MZ-vs-DZ strain-sharing patterns.

## When NOT to use

- Strains cannot be resolved because samples were compared with species-level abundance tools (e.g. MetaPhlAn) rather than inStrain popANI comparisons
- The question is within-sample microdiversity or nucleotide diversity (e.g. Klebsiella diversity by birth mode), not between-sample strain sharing
- Samples lack relationship metadata (sibling/MZ/DZ status, birth day, birth weight, gestational age), making the enrichment and correlation tests underpowered

## Inputs

- inStrain profile objects for each infant sample (reads mapped to concatenated dereplicated genome set via Bowtie 2)
- inStrain compare output of pairwise popANI and coverage-overlap values
- Infant metadata table: sibling/MZ/DZ relationships, birth day, birth weight, gestational age

## Outputs

- Strain identities per infant (analogous to Supplemental Table S4)
- Strain-by-infant presence/absence matrix and pairwise Jaccard similarity matrix
- Fisher's exact test p-value for sibling-sharing enrichment of two-infant strains
- Mann-Whitney/rank-sum statistic comparing MZ vs DZ strain sharing
- Figure 3-style panels (Matplotlib/Seaborn): sibling enrichment, MZ-vs-DZ, physiological-property associations

## How to apply

Run 'inStrain compare' on all pairs of inStrain profile objects (built from reads mapped to a concatenated dereplicated genome set, de-replicated at ~98% ANI with dRep) to obtain pairwise popANI and coverage-overlap values. Cluster sub-species into strains using 'average' hierarchical clustering with a 99.999% ANI threshold, then define detection per infant as presence at >= 0.5 unmaskedBreadth and build a strain-by-infant presence/absence matrix. Compute pairwise Jaccard similarity of strain composition between infants, and for strains present in exactly two infants use a SciPy Fisher's exact test to test sibling-sharing enrichment. Compare MZ vs DZ sharing with a SciPy rank-sum (Mann-Whitney) test, and correlate Jaccard similarity with birth-day distance, birth-weight difference, and gestational-age difference to reproduce Figure 3 panels. The rationale is that popANI-based strain clustering resolves strains below the 98% ANI subspecies level, enabling sensitive detection of shared strains that species-level methods miss.

## Related tools

- **inStrain** (Profiles each infant sample and runs 'inStrain compare' to obtain pairwise popANI and coverage-overlap values used to cluster strains) — github.com/MrOlm/instrain
- **dRep** (De-replicates the genome set (~98% ANI) into representative genomes for the concatenated reference used in mapping) — github.com/MrOlm/drep
- **SciPy** (Fisher's exact test for sibling enrichment and rank-sum (Mann-Whitney) test for MZ vs DZ sharing)
- **Pandas** (Builds the strain-by-infant presence/absence matrix and Jaccard similarity table)
- **Numpy** (Numerical handling of similarity matrices during statistical testing)
- **Matplotlib** (Draws the Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological associations))
- **Seaborn** (Statistical plotting of similarity-vs-property associations)

## Examples

```
$ inStrain compare inStrain_profile1 inStrain_profile2 -o compare_output
```

## Evaluation signals

- Strains present in exactly two infants show a statistically significant sibling-sharing enrichment by Fisher's exact test (matching Figure 3)
- MZ twins share no more bacterial, bacteriophage, or plasmid strains than DZ twins (rank-sum test not significant)
- Jaccard similarity increases with birth-day proximity and with similarity of gestational age and birth weight
- Majority of identified strains are found in only a single infant, as a sanity check on strain granularity
- Strain identities per infant reproduce the structure of Supplemental Table S4 from 99.999% ANI average-linkage clustering

## Limitations

- It is difficult to distinguish microbiome diversity that evolved in situ from that introduced by immigration
- Prior twin studies are subject to confounding because diet and lifestyle preferences are more similar in MZ twins than DZ twins
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads; variant sites near the ends of genome fragments can be missed with inStrain
- Raw comparison values and strain identities are only in Supplemental Table S4, not the main text

## Evidence

- [readme] InStrain compare usage: [evidence span withheld — anchor status: external_doc]
