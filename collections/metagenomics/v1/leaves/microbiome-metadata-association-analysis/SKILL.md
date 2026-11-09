---
name: microbiome-metadata-association-analysis
description: Use when you have per-sample inStrain profile objects (or equivalent
  strain calls) plus per-infant metadata (sibling/twin relationships, birth day, birth
  weight, gestational age) and need to test whether strain sharing correlates with
  these host properties.
when_to_use_negative:
- You only need species-level abundance association (e.g. differential abundance of
  taxa vs metadata) rather than sub-species strain-sharing inference — inStrain popANI/strain
  clustering is unnecessary.
- Your samples lack paired metadata such as kinship (siblings/MZ vs DZ), birth day,
  birth weight, or gestational age, so the Fisher's exact and rank-sum tests cannot
  be framed.
- Strains are called from reference genomes at species-level ANI thresholds rather
  than 99.999% ANI strain clusters, making the two-infant strain-sharing design inapplicable.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_0203
tools:
- name: inStrain
  role: Produces the profile objects and pairwise popANI/coverage-overlap comparisons
    from .bam files that feed the strain-sharing analysis
  repo: github.com/MrOlm/instrain
- name: SciPy
  role: Fisher's exact test for sibling enrichment of two-infant strains and rank-sum
    (Mann-Whitney) test for MZ-vs-DZ sharing
- name: Pandas
  role: Builds the strain-by-infant presence/absence matrix and merges infant metadata
    with strain identities
- name: Numpy
  role: Numeric handling of Jaccard similarity and correlation computations
- name: Matplotlib
  role: Draws the Figure 3-style association panels
- name: Seaborn
  role: Statistical plotting of sibling-enrichment, MZ-vs-DZ, and physiological-property
    associations
- name: dRep
  role: Dereplicates the genome set (98% ANI) before read mapping so inStrain profiles
    are built on representative sub-species genomes
  repo: github.com/MrOlm/drep
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:d89b17621fe4170008d0c2b03d70e4db0bb513c1c773c0eefde36be5848b92fa
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# microbiome-metadata-association-analysis

## Summary

Tests whether strain-level microbial sharing among NICU infants (detected with inStrain popANI comparisons and 99.999% ANI strain clustering) is associated with host metadata — sibling relationship, twin zygosity (MZ vs DZ), birth-day distance, birth weight, and gestational age — using SciPy statistical tests and Figure 3-style visualizations.

## When to use

Use when you have per-sample inStrain profile objects (or equivalent strain calls) plus per-infant metadata (sibling/twin relationships, birth day, birth weight, gestational age) and need to test whether strain sharing correlates with these host properties.

## When NOT to use

- You only need species-level abundance association (e.g. differential abundance of taxa vs metadata) rather than sub-species strain-sharing inference — inStrain popANI/strain clustering is unnecessary.
- Your samples lack paired metadata such as kinship (siblings/MZ vs DZ), birth day, birth weight, or gestational age, so the Fisher's exact and rank-sum tests cannot be framed.
- Strains are called from reference genomes at species-level ANI thresholds rather than 99.999% ANI strain clusters, making the two-infant strain-sharing design inapplicable.

## Inputs

- inStrain profile objects built from each infant sample (reads mapped to a concatenated dereplicated genome set, e.g. dereplicated at 98% ANI with dRep)
- Pairwise popANI and coverage-overlap table from 'inStrain compare'
- Infant metadata table: sibling/twin (MZ/DZ) relationships, birth day, birth weight, gestational age
- Supplemental-Table-S4-style strain identities per infant

## Outputs

- Strain-by-infant binary presence/absence matrix (detection = >= 0.5 unmaskedBreadth)
- Pairwise Jaccard similarity matrix of strain composition between infants
- Fisher's exact test result for sibling enrichment of two-infant strains
- Mann-Whitney (rank-sum) result comparing MZ vs DZ twin strain sharing
- Correlation statistics between Jaccard similarity and birth-day distance, birth-weight difference, and gestational-age difference
- Figure 3-style panels (sibling enrichment, MZ-vs-DZ, physiological-property associations)

## How to apply

Run 'inStrain compare' on all pairs of inStrain profile objects to obtain pairwise popANI and coverage-overlap values, then cluster sub-species into strains using 'average' hierarchical clustering with a threshold of 99.999% ANI. Define detection of a sub-species in a sample as that genome being present with >= 0.5 unmaskedBreadth, build a strain-by-infant presence/absence matrix, and compute pairwise Jaccard similarity of strain composition between all infants. For strains found in exactly two infants, use a SciPy Fisher's exact test to test whether two-infant strains are shared between siblings more often than expected by chance; use a SciPy rank-sum (Mann-Whitney) test to compare MZ vs DZ sharing. Correlate Jaccard similarity with birth-day distance, birth-weight difference, and gestational-age difference, and draw the sibling-enrichment, MZ-vs-DZ, and physiological-property panels with Matplotlib/Seaborn. Rationale: strain identity (not just species presence) is what resolves transmission, and the paper showed the majority of strains occur in only a single infant, so association tests must focus on strains shared between exactly two individuals.

## Related tools

- **inStrain** (Produces the profile objects and pairwise popANI/coverage-overlap comparisons from .bam files that feed the strain-sharing analysis) — github.com/MrOlm/instrain
- **SciPy** (Fisher's exact test for sibling enrichment of two-infant strains and rank-sum (Mann-Whitney) test for MZ-vs-DZ sharing)
- **Pandas** (Builds the strain-by-infant presence/absence matrix and merges infant metadata with strain identities)
- **Numpy** (Numeric handling of Jaccard similarity and correlation computations)
- **Matplotlib** (Draws the Figure 3-style association panels)
- **Seaborn** (Statistical plotting of sibling-enrichment, MZ-vs-DZ, and physiological-property associations)
- **dRep** (Dereplicates the genome set (98% ANI) before read mapping so inStrain profiles are built on representative sub-species genomes) — github.com/MrOlm/drep

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Two-infant strains show a statistically significant sibling-sharing enrichment in the Fisher's exact test, reproducing the paper's finding that 'Strains present in two and only two infants were significantly more likely to be shared between siblings (Figure 3)'.
- MZ twin pairs share no more strains than DZ pairs (Mann-Whitney test not significant), matching the reported result that MZ twins shared no more strains of bacteria, bacteriophage, or plasmids than DZ twins.
- Jaccard similarity increases with birth-day proximity and physiological similarity (gestational age, birth weight), as reported for infants with similar gestational age and birth weight.
- Strain calls derive from 'average' hierarchical clustering at a 99.999% ANI threshold and detection uses >= 0.5 unmaskedBreadth — verify these parameters in the pipeline outputs.
- Sanity check: the majority of strains should appear in only a single infant; a strain matrix where most strains are multi-infant indicates a detection-threshold or contamination problem (e.g. control-sample strains not flagged).

## Limitations

- It is difficult to distinguish microbiome diversity that evolved in situ from that introduced by immigration, so shared strains may reflect transmission rather than shared environmental exposure alone.
- MZ twins may confound zygosity tests: diet and lifestyle preferences are more similar in MZ twins than DZ twins, presenting significant potential for confounding effects.
- Sub-species detected in the negative extraction control sample, or significantly enriched in one sampling campaign, are marked 'likely contaminant' and must be excluded before association testing.

## Evidence

- [readme] Pairwise comparison command: [evidence span withheld — anchor status: external_doc]
