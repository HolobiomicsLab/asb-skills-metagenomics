---
name: infant-strain-sharing-jaccard-similarity-analysis
description: Use when metagenomic samples from related individuals (e.g. NICU infants,
  twin pairs) have inStrain profiles and the goal is to quantify and statistically
  test which pairs share microbial strains.
when_to_use_negative:
- The question is strain detection accuracy or SNV calling from a single sample rather
  than between-infant sharing statistics
- Samples are unrelated and lack relationship metadata (siblings/twins), so the sibling
  and MZ-vs-DZ tests cannot be constructed
- Genome set has not been de-replicated (e.g. with dRep at ~98% ANI), leading to multi-mapped
  reads that confound comparisons
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0194
tools:
- name: inStrain
  role: Profiles each infant sample and runs 'inStrain compare' to produce pairwise
    popANI and coverage-overlap values used to define strains and sharing
  repo: github.com/MrOlm/instrain
- name: Pandas
  role: Build the strain-by-infant presence/absence matrix and compute Jaccard similarity
    between infants
- name: SciPy
  role: Fisher's exact test for sibling sharing enrichment and rank-sum (Mann-Whitney)
    test for MZ vs DZ comparison
- name: Numpy
  role: Numerical handling of similarity matrices and test statistics
- name: Matplotlib
  role: Draw the Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property
    associations)
- name: Seaborn
  role: Statistical plotting of Figure 3-style association panels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:7e35a8e9ed72e4753d92654d1e061c7d33df247089a540a6bb95ad4a40c997fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# infant-strain-sharing-jaccard-similarity-analysis

## Summary

Reproduce the NICU Figure 3 strain-sharing analysis: cluster inStrain sub-species into strains (99.999% ANI), build a strain-by-infant presence/absence matrix, compute pairwise Jaccard similarity, and statistically test sibling enrichment, MZ-vs-DZ sharing, and associations with birth-day distance, birth weight, and gestational age.

## When to use

Use when metagenomic samples from related individuals (e.g. NICU infants, twin pairs) have inStrain profiles and the goal is to quantify and statistically test which pairs share microbial strains.

## When NOT to use

- The question is strain detection accuracy or SNV calling from a single sample rather than between-infant sharing statistics
- Samples are unrelated and lack relationship metadata (siblings/twins), so the sibling and MZ-vs-DZ tests cannot be constructed
- Genome set has not been de-replicated (e.g. with dRep at ~98% ANI), leading to multi-mapped reads that confound comparisons

## Inputs

- inStrain profile objects for each infant sample (from reads mapped to concatenated dereplicated genomes)
- inStrain compare output tables with pairwise popANI and coverage-overlap values
- Infant metadata table: sibling/twin relationships, birth day, birth weight, gestational age
- Strain identity assignments (Supplemental Table S4 analog)

## Outputs

- Strain-by-infant presence/absence matrix (detection = >= 0.5 unmaskedBreadth)
- Pairwise infant Jaccard similarity matrix of strain composition
- Fisher's exact test p-value for sibling sharing enrichment of two-infant strains
- Mann-Whitney/rank-sum comparison of MZ vs DZ twin strain sharing
- Correlations of Jaccard similarity with birth-day distance, birth-weight difference, and gestational-age difference
- Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property associations)

## How to apply

Run 'inStrain compare' on all pairs of inStrain profile objects (built from reads mapped with Bowtie 2 to the concatenated dereplicated genome set) to obtain pairwise popANI and coverage-overlap values. Cluster sub-species into individual strains using 'average' hierarchical clustering with a threshold of 99.999% ANI and record strain identities per infant (Supplemental Table S4 analog). Define detection of a sub-species in a sample as that genome being present with >= 0.5 unmaskedBreadth, then build a strain-by-infant presence/absence matrix and compute pairwise Jaccard similarity between all infants. For strains found in exactly two infants, use SciPy's Fisher's exact test to test whether strains are more likely shared between siblings than expected by chance, and SciPy's rank-sum (Mann-Whitney) test to compare MZ vs DZ sharing. Correlate Jaccard similarity with birth-day distance, birth-weight difference, and gestational-age difference, and draw the Figure 3 panels with Matplotlib/Seaborn.

## Related tools

- **inStrain** (Profiles each infant sample and runs 'inStrain compare' to produce pairwise popANI and coverage-overlap values used to define strains and sharing) — github.com/MrOlm/instrain
- **Pandas** (Build the strain-by-infant presence/absence matrix and compute Jaccard similarity between infants)
- **SciPy** (Fisher's exact test for sibling sharing enrichment and rank-sum (Mann-Whitney) test for MZ vs DZ comparison)
- **Numpy** (Numerical handling of similarity matrices and test statistics)
- **Matplotlib** (Draw the Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property associations))
- **Seaborn** (Statistical plotting of Figure 3-style association panels)

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Strains present in exactly two infants show a statistically significant enrichment for being shared between siblings (Fisher's exact test)
- MZ twins share no more strains of bacteria, bacteriophage, or plasmids than DZ twins (rank-sum test not significant in the MZ-favoring direction)
- Jaccard similarity increases with birth-day proximity and with physiological similarity (gestational age, birth weight)
- Strain identities reproduce the Supplemental Table S4 structure using 'average' hierarchical clustering at 99.999% ANI
- Detection calls are consistent with the >= 0.5 unmaskedBreadth definition, with the majority of strains appearing in only a single infant

## Limitations

- It is difficult to distinguish microbiome diversity that evolved in situ from that introduced by immigration
- Prior twin studies are subject to confounding because diet and lifestyle preferences are more similar in MZ than DZ twins
- Raw comparison values and strain identities live in Supplemental Table S4; sub-species information in Supplemental Table S5, which are not included in the main text
- Variant sites near the ends of genome fragments can be missed with inStrain, potentially affecting strain calls

## Evidence

- [readme] inStrain compare invocation: [evidence span withheld — anchor status: external_doc]
