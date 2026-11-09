---
name: multi-panel-figure-visualization
description: Use when metagenomic samples (e.g., NICU infant stool) have been profiled
  with inStrain and compared pairwise, and you need to test and visualize whether
  shared strains are enriched between siblings, whether MZ twins share more than DZ
  twins, and whether similarity tracks physiological properties.
when_to_use_negative:
- You only have single-sample inStrain profiles (nucleotide diversity, SNVs, linkage)
  and no pairwise comparisons between infants — run 'inStrain compare' first; this
  skill operates on cross-sample sharing statistics
- Samples lack paired metadata (sibling/twin status, birth day, birth weight, gestational
  age), so the sibling-enrichment, MZ-vs-DZ, and physiology panels cannot be populated
- The goal is a generic single-panel plot of one metric rather than the reproducible
  multi-panel strain-sharing analysis with its associated statistical tests
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: inStrain
  role: Generates the profile objects and the 'inStrain compare' popANI/coverage-overlap
    values that feed strain clustering and the strain-by-infant matrix
  repo: github.com/MrOlm/instrain
- name: Pandas
  role: Assembles the strain-by-infant presence/absence matrix, Jaccard similarity
    matrix, and merged infant metadata for plotting
- name: SciPy
  role: Provides Fisher's exact test (two-infant strain sibling sharing), rank-sum/Mann-Whitney
    test (MZ vs DZ), and correlation statistics behind the panels
- name: Numpy
  role: Numerical array handling for the similarity matrices and test statistics
- name: Matplotlib
  role: Renders the Figure 3-style multi-panel figure
- name: Seaborn
  role: Statistical plotting layer for the enrichment, twin-comparison, and association
    panels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:1b855baa0c185fffe620687729f15db1ebce11885634ca2fb51fefa385ccdf35
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-panel-figure-visualization

## Summary

Reproduces the Figure 3-style multi-panel analysis of strain-sharing among NICU infants: sibling enrichment of two-infant strains, MZ-vs-DZ twin comparison, and association of strain-composition Jaccard similarity with birth-day distance, birth weight, and gestational age, drawn with Matplotlib/Seaborn on top of inStrain compare outputs.

## When to use

Use when metagenomic samples (e.g., NICU infant stool) have been profiled with inStrain and compared pairwise, and you need to test and visualize whether shared strains are enriched between siblings, whether MZ twins share more than DZ twins, and whether similarity tracks physiological properties.

## When NOT to use

- You only have single-sample inStrain profiles (nucleotide diversity, SNVs, linkage) and no pairwise comparisons between infants — run 'inStrain compare' first; this skill operates on cross-sample sharing statistics
- Samples lack paired metadata (sibling/twin status, birth day, birth weight, gestational age), so the sibling-enrichment, MZ-vs-DZ, and physiology panels cannot be populated
- The goal is a generic single-panel plot of one metric rather than the reproducible multi-panel strain-sharing analysis with its associated statistical tests

## Inputs

- inStrain profile objects for each infant sample (built from .bam files of reads mapped to the concatenated dereplicated genome set)
- Pairwise comparison table from 'inStrain compare' (popANI and coverage-overlap values)
- Infant metadata table: sibling/twin (MZ/DZ) relationships, birth day, birth weight, gestational age
- Negative extraction control and sampling-campaign information for flagging 'likely contaminant' sub-species before analysis

## Outputs

- Strain identities per infant and strain-by-infant presence/absence matrix (Supplemental Table S4 analogue)
- Pairwise Jaccard similarity matrix of infant strain composition
- Statistical test results: Fisher's exact test on two-infant strain sibling sharing, Mann-Whitney rank-sum for MZ vs DZ, correlations with birth-day distance, birth-weight difference, and gestational-age difference
- Figure 3-style multi-panel plots (sibling enrichment, MZ-vs-DZ, physiological-property associations) rendered with Matplotlib/Seaborn

## How to apply

Run 'inStrain compare' on the inStrain profile objects built from reads mapped to the concatenated dereplicated genome set (dereplicated at 98% ANI to reduce multi-mapped reads) to obtain all pairwise popANI and coverage-overlap values. Cluster sub-species into strains with 'average' hierarchical clustering at a 99.999% ANI threshold, and define a sub-species as detected in an infant when present at >= 0.5 unmaskedBreadth, yielding a strain-by-infant presence/absence matrix (analogous to Supplemental Table S4). From that matrix compute pairwise Jaccard similarity of strain composition between infants, restrict two-infant strains for the sibling test, and test sharing enrichment with a SciPy Fisher's exact test. Compare MZ vs DZ twin-pair similarity with a SciPy rank-sum (Mann-Whitney) test, and correlate Jaccard similarity with birth-day distance, birth-weight difference, and gestational-age difference. Render the three Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property associations) with Matplotlib/Seaborn, using Pandas/Numpy to assemble the matrices.

## Related tools

- **inStrain** (Generates the profile objects and the 'inStrain compare' popANI/coverage-overlap values that feed strain clustering and the strain-by-infant matrix) — github.com/MrOlm/instrain
- **Pandas** (Assembles the strain-by-infant presence/absence matrix, Jaccard similarity matrix, and merged infant metadata for plotting)
- **SciPy** (Provides Fisher's exact test (two-infant strain sibling sharing), rank-sum/Mann-Whitney test (MZ vs DZ), and correlation statistics behind the panels)
- **Numpy** (Numerical array handling for the similarity matrices and test statistics)
- **Matplotlib** (Renders the Figure 3-style multi-panel figure)
- **Seaborn** (Statistical plotting layer for the enrichment, twin-comparison, and association panels)

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Strains present in two and only two infants show a statistically significant Fisher's-exact enrichment for sharing between siblings, matching the reported Figure 3 pattern
- MZ twin-pair similarity does not exceed DZ twin-pair similarity in the rank-sum (Mann-Whitney) test — the expected null result for MZ excess sharing
- Jaccard similarity increases with birth-day proximity and physiological similarity (smaller birth-weight and gestational-age differences), consistent with the reported associations
- Strain calls derive from 'average' hierarchical clustering at 99.999% ANI and detection is thresholded at >= 0.5 unmaskedBreadth; sub-species found in the negative extraction control or enriched in one campaign are excluded as 'likely contaminant'
- The majority of strains in the presence/absence matrix appear in only a single infant, matching the reported distribution

## Limitations

- It is difficult to distinguish microbiome diversity that evolved in situ from diversity introduced by immigration, which constrains interpretation of observed sharing patterns
- Prior twin-study designs are subject to confounding because diet and lifestyle preferences are more similar in MZ than DZ twins, presenting potential confounding effects on the MZ-vs-DZ comparison
- Raw comparison values and strain identities were deferred to Supplemental Table S4, so figure reproduction depends on access to those tables or regenerated 'inStrain compare' output
- Variant sites near the ends of genome fragments can be missed with inStrain, which can slightly affect strain-detection calls feeding the matrix

## Evidence

- [readme] Compare command producing pairwise inputs: [evidence span withheld — anchor status: external_doc]
