---
name: ani-threshold-strain-sharing-analysis
description: Use when the goal is to decide, from metagenomic read mappings to a set
  of sub-species genomes, which strains are shared between samples, and to compare
  strain-sharing calls across ANI thresholds (e.g., 99.999% strain clustering vs.
  98% dereplication).
when_to_use_negative:
- Input is only assemblies/reference genomes with no sample read mappings — ANI dereplication
  alone (dRep, FastANI) suffices and popANI read-based microdiversity comparisons
  are not possible
- The question is within-sample microdiversity, SNV calling, or linkage rather than
  between-sample strain sharing
- Species-level presence/absence profiling is the goal, and strain-level (99.999%
  ANI) resolution is not required
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Profiles each sample from the .bam mapping and performs pairwise popANI comparisons
    between profile objects; provides unmaskedBreadth for the 0.5 detection criterion
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps each sample's reads to the concatenated sub-species genome .fasta with
    default settings to produce the .bam inputs for inStrain profile
- name: dRep
  role: Dereplicates the same genome set at a 98% ANI threshold to provide an ANI-based
    shared-genome benchmark count
  repo: https://github.com/MrOlm/drep
- name: Matplotlib
  role: Renders the per-ANI-threshold shared-genome counts as the Figure 2d comparison
    plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:3400e1cae95336fc742efb7fac2e372b6a6439bb1500af16f2e742e0881bec90
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Bowtie2
  - Matplotlib
  - Pandas
  - dRep
  - inStrain
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ani-threshold-strain-sharing-analysis

## Summary

Determines which microbial strains are shared between samples by mapping reads to sub-species genome sets, computing microdiversity-aware popANI comparisons with inStrain, clustering subspecies into strains at a 99.999% ANI threshold, and benchmarking the shared-genome counts against ANI-based tools (dRep, MIDAS, StrainPhlAn).

## When to use

Use when the goal is to decide, from metagenomic read mappings to a set of sub-species genomes, which strains are shared between samples, and to compare strain-sharing calls across ANI thresholds (e.g., 99.999% strain clustering vs. 98% dereplication).

## When NOT to use

- Input is only assemblies/reference genomes with no sample read mappings — ANI dereplication alone (dRep, FastANI) suffices and popANI read-based microdiversity comparisons are not possible
- The question is within-sample microdiversity, SNV calling, or linkage rather than between-sample strain sharing
- Species-level presence/absence profiling is the goal, and strain-level (99.999% ANI) resolution is not required

## Inputs

- Bowtie2 .bam read mappings of each sample against a single concatenated sub-species genome .fasta
- Deposited genome bins (e.g., infant genome bins from twin sets)
- Paired-end metagenomic read sets (same reads used for all tools)

## Outputs

- supplemental_table_s1.csv with per-tool shared-genome counts at each ANI threshold
- Figure 2d-style comparison of shared-genome counts across inStrain, dRep, MIDAS, and StrainPhlAn
- inStrain 'compare' output tables with pairwise popANI and coverage overlap values

## How to apply

Run 'inStrain profile' on each sample's Bowtie2 mapping against a single concatenated sub-species genome .fasta, then run 'inStrain compare' pairwise on the resulting profile objects to obtain popANI and coverage overlap for every sample pair (the microdiversity-aware comparison that dramatically increases genomic comparison accuracy relative to prior strain-comparison methods). Define a strain as detected in a sample when the sub-species genome is present with >= 0.5 unmaskedBreadth, and cluster subspecies into individual strains using 'average' hierarchical clustering with a threshold of 99.999% ANI. In parallel, run dRep on the same genome set dereplicating at a 98% ANI threshold (98% or lower is recommended to reduce multi-mapped reads), and run MIDAS and StrainPhlAn on the same read sets to count shared genomes under each tool's native strain-sharing criterion. Tabulate the number of genomes shared at each ANI threshold for each tool and render the comparison (as Figure 2d in the infant twin-cohort application), writing raw counts to supplemental_table_s1.csv.

## Related tools

- **inStrain** (Profiles each sample from the .bam mapping and performs pairwise popANI comparisons between profile objects; provides unmaskedBreadth for the 0.5 detection criterion) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps each sample's reads to the concatenated sub-species genome .fasta with default settings to produce the .bam inputs for inStrain profile)
- **dRep** (Dereplicates the same genome set at a 98% ANI threshold to provide an ANI-based shared-genome benchmark count) — https://github.com/MrOlm/drep
- **Matplotlib** (Renders the per-ANI-threshold shared-genome counts as the Figure 2d comparison plot)

## Examples

```
inStrain compare inStrain_profile1 inStrain_profile2 -o compare_output
```

## Evaluation signals

- supplemental_table_s1.csv exists and contains per-tool shared-genome counts at each ANI threshold
- inStrain shared-genome counts agree with the values reported in the paper's Supplemental Table S1 and Figure 2d
- A genome is only called shared when it is detected in both samples with >= 0.5 unmaskedBreadth
- Strain clusters are formed with 'average' hierarchical clustering at a threshold of 99.999% ANI
- dRep dereplication is performed at a 98% ANI threshold (or lower) on the same genome set used for inStrain

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads before comparison
- Raw benchmarking values are deferred to Supplemental Table S1 and are not included in the main text
- inStrain's Python implementation is slow (the changelog notes it should call the node2vec C++ library in the future)

## Evidence

- [discussion] cluster subspecies into a number of individual strains using ‘average’ hierarchical clustering with a threshold of 99.999% ANI: "cluster subspecies into a number of individual strains using ‘average’ hierarchical clustering with a threshold of 99.999% ANI"
- [discussion] Detection of a sub-species in a sample was defined as that genome being present with >= 0.5 unmaskedBreadth: "Detection of a sub-species in a sample was defined as that genome being present with >= 0.5 unmaskedBreadth"
- [methods] when dereplicating your genome set (using `dRep <https://github.com/MrOlm/drep>`_ for example), using a threshold of 98% or lower is a good idea.: "when dereplicating your genome set (using `dRep <https://github.com/MrOlm/drep>`_ for example), using a threshold of 98% or lower is a good idea."
- [discussion] InStrain performs strain-level comparisons by comparing inStrain profile objects that were created by mapping different sets of reads: "InStrain performs strain-level comparisons by comparing inStrain profile objects that were created by mapping different sets of reads"
- [readme] inStrain compare inStrain_profile1 inStrain_profile2: "### Detailed strain-level comparison:
``n$ inStrain compare inStrain_profile1 inStrain_profile2
"
