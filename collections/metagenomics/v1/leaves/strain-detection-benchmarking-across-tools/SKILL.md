---
name: strain-detection-benchmarking-across-tools
description: Use when the goal is to compare how many microbial strains are shared
  between samples (e.g., twin siblings) under different strain-comparison criteria,
  or to validate inStrain's popANI approach against dRep/MIDAS/StrainPhlAn on the
  same mappings.
when_to_use_negative:
- Input is a single sample or a species-level abundance question — this skill addresses
  sub-species strain sharing, not presence/absence at species rank.
- Read mappings are to heterogeneous genome sets dereplicated above 98% ANI; multi-mapped
  reads will confound the comparison (the paper notes de-replication at 98% ANI reduces
  multi-mapped reads).
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Profiles each sample's mapping (microdiversity, SNVs, breadth) and performs
    pairwise popANI/ConANI comparisons used as the microdiversity-aware strain-sharing
    criterion
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps each sample's reads to all representative sub-species genomes concatenated
    together with default settings, producing the .bam inputs
- name: dRep
  role: Dereplicates the same genome set at a 98% ANI threshold to count genomes shared
    between infants under an ANI-based criterion
  repo: https://github.com/MrOlm/drep
- name: Pandas
  role: Tabulates per-tool shared-genome counts at each ANI threshold into supplemental_table_s1.csv
- name: Matplotlib
  role: Renders the per-tool shared-genome comparison plot analogous to Figure 2d
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:29d4bcbfcd6ffe6b6099c3ca71309c5256d35141a72e41d5bcad757b10040131
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# strain-detection-benchmarking-across-tools

## Summary

Benchmark strain-sharing detection by comparing inStrain's microdiversity-aware popANI-based comparisons against dRep, MIDAS, and StrainPhlAn on the same infant twin metagenomic read sets, tabulating genomes shared at each ANI threshold per tool.

## When to use

Use when the goal is to compare how many microbial strains are shared between samples (e.g., twin siblings) under different strain-comparison criteria, or to validate inStrain's popANI approach against dRep/MIDAS/StrainPhlAn on the same mappings.

## When NOT to use

- Input is a single sample or a species-level abundance question — this skill addresses sub-species strain sharing, not presence/absence at species rank.
- Read mappings are to heterogeneous genome sets dereplicated above 98% ANI; multi-mapped reads will confound the comparison (the paper notes de-replication at 98% ANI reduces multi-mapped reads).

## Inputs

- Infant metagenomic read sets from three twin sets
- Deposited infant genome bins (sub-species genomes) concatenated into a single .fasta
- Bowtie2 .bam mappings of each sample's reads to the concatenated reference .fasta
- inStrain profile output directories (one per sample) for 'inStrain compare'

## Outputs

- supplemental_table_s1.csv with per-tool shared-genome counts at each ANI threshold
- Figure 2d-style comparison plot of genomes shared per tool (Matplotlib)
- inStrain compare pairwise popANI and coverage-overlap table

## How to apply

Run 'inStrain profile' on each sample's Bowtie2 .bam mapping to the same concatenated sub-species genome .fasta, then run 'inStrain compare' pairwise to compute popANI and coverage overlap between all sample pairs. Define a strain as shared when a sub-species genome is detected in both infants with >= 0.5 unmaskedBreadth, and cluster subspecies into strains using 'average' hierarchical clustering at a 99.999% ANI threshold. On the same genome set, run dRep dereplicating at a 98% ANI threshold (inStrain's methods recommend 98% or lower for dereplication to reduce multi-mapped reads), plus MIDAS and StrainPhlAn under each tool's native strain-sharing criterion. Tabulate the number of genomes shared at each ANI threshold for each of the four tools and render the comparison (as in the paper's Figure 2d), writing raw counts to supplemental_table_s1.csv. Validation: verify supplemental_table_s1.csv exists with per-tool shared-genome counts and that the inStrain counts agree with the paper's Supplemental Table S1 and Figure 2d.

## Related tools

- **inStrain** (Profiles each sample's mapping (microdiversity, SNVs, breadth) and performs pairwise popANI/ConANI comparisons used as the microdiversity-aware strain-sharing criterion) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps each sample's reads to all representative sub-species genomes concatenated together with default settings, producing the .bam inputs)
- **dRep** (Dereplicates the same genome set at a 98% ANI threshold to count genomes shared between infants under an ANI-based criterion) — https://github.com/MrOlm/drep
- **Pandas** (Tabulates per-tool shared-genome counts at each ANI threshold into supplemental_table_s1.csv)
- **Matplotlib** (Renders the per-tool shared-genome comparison plot analogous to Figure 2d)

## Examples

```
inStrain profile mapping.bam genomes.fasta -o inStrain_profile1; inStrain compare inStrain_profile1 inStrain_profile2; dRep dereplicate drep_out -g genomes/*.fasta --S_ani 0.98
```

## Evaluation signals

- supplemental_table_s1.csv exists and contains per-tool shared-genome counts at each ANI threshold for inStrain, dRep, MIDAS, and StrainPhlAn
- inStrain counts agree with the values reported in the paper's Supplemental Table S1 and Figure 2d
- Every pairwise inStrain comparison was computed between profile objects generated from the same concatenated .fasta reference (no cross-reference comparisons)
- Strain-sharing counts use the stated criteria: >= 0.5 unmaskedBreadth detection and 'average' hierarchical clustering at 99.999% ANI
- dRep dereplication used a 98% ANI threshold, matching the paper's recommendation for reducing multi-mapped reads

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 and are not in the main text
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads; skipping it biases shared-genome counts
- Variant sites near the ends of genome fragments can be missed with inStrain, which may affect SNV-based strain criteria
- inStrain's Python implementation is slow (changelog notes node2vec C++ library should be called in the future)

## Evidence
