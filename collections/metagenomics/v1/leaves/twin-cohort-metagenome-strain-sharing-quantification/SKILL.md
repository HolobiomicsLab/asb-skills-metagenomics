---
name: twin-cohort-metagenome-strain-sharing-quantification
description: Use when quantifying microbial strains shared between related individuals
  (e.g., twin pairs) from metagenomic read mappings, or when benchmarking a strain-comparison
  method's accuracy against existing ANI-based tools.
when_to_use_negative:
- Input is raw reads without a curated set of sub-species reference genomes to concatenate
  and map against — inStrain profile requires a .fasta of representative genomes
- The question is species-level abundance profiling rather than sub-species strain
  sharing — tools like CoverM for genome coverage would be more appropriate
- Genomes were dereplicated at >98% ANI, which risks multi-mapped reads and inflated
  comparisons
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: inStrain
  role: Profiles each sample's mapping and performs microdiversity-aware pairwise
    comparisons (popANI, unmaskedBreadth, coverage overlap) to call shared strains
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps each sample's reads to the concatenated sub-species genome .fasta with
    default settings to produce the .bam inputs
- name: dRep
  role: 'Comparator: counts genomes shared between infants by ANI-based dereplication
    criterion at a 98% threshold'
  repo: https://github.com/MrOlm/drep
- name: Pandas
  role: Tabulates per-tool shared-genome counts and writes supplemental_table_s1.csv
- name: Matplotlib
  role: Renders the tool-comparison figure (Figure 2d)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:4161fa16104900f877fdca2256f4029beadb8b1511eae2cdb8d8217d3d5ec711
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# twin-cohort-metagenome-strain-sharing-quantification

## Summary

Benchmarks strain-sharing detection across infant twin metagenomes by quantifying genomes shared between siblings using inStrain's microdiversity-aware popANI comparisons against dRep, MIDAS, and StrainPhlAn native criteria.

## When to use

Use when quantifying microbial strains shared between related individuals (e.g., twin pairs) from metagenomic read mappings, or when benchmarking a strain-comparison method's accuracy against existing ANI-based tools.

## When NOT to use

- Input is raw reads without a curated set of sub-species reference genomes to concatenate and map against — inStrain profile requires a .fasta of representative genomes
- The question is species-level abundance profiling rather than sub-species strain sharing — tools like CoverM for genome coverage would be more appropriate
- Genomes were dereplicated at >98% ANI, which risks multi-mapped reads and inflated comparisons

## Inputs

- Bowtie2 mappings (.bam) of each infant sample's reads to all representative sub-species genomes concatenated into a single .fasta
- Deposited infant genome bins from the three twin sets
- Same read sets for MIDAS and StrainPhlAn comparators
- Genome set for dRep dereplication

## Outputs

- supplemental_table_s1.csv with per-tool shared-genome counts at each ANI threshold
- Figure 2d comparison of shared-genome counts across inStrain, dRep, MIDAS, and StrainPhlAn
- inStrain compare output with pairwise popANI and coverage overlap values

## How to apply

Run 'inStrain profile' on each sample's Bowtie2 mapping (.bam) against the concatenated sub-species genome .fasta, then run 'inStrain compare' pairwise to compute popANI and coverage overlap. Define a strain as shared when a sub-species genome is detected in both infants with >= 0.5 unmaskedBreadth, and cluster subspecies into strains using 'average' hierarchical clustering at a 99.999% ANI threshold. As comparators, run dRep (dereplicating at a 98% ANI threshold, which also reduces multi-mapped reads), MIDAS, and StrainPhlAn on the same read sets under their native strain-sharing criteria. Tabulate genomes shared at each ANI threshold per tool, then verify the inStrain counts agree with the paper's Supplemental Table S1 and Figure 2d.

## Related tools

- **inStrain** (Profiles each sample's mapping and performs microdiversity-aware pairwise comparisons (popANI, unmaskedBreadth, coverage overlap) to call shared strains) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps each sample's reads to the concatenated sub-species genome .fasta with default settings to produce the .bam inputs)
- **dRep** (Comparator: counts genomes shared between infants by ANI-based dereplication criterion at a 98% threshold) — https://github.com/MrOlm/drep
- **Pandas** (Tabulates per-tool shared-genome counts and writes supplemental_table_s1.csv)
- **Matplotlib** (Renders the tool-comparison figure (Figure 2d))

## Examples

```
inStrain profile mapping.bam genomes.fasta -o ISP1; inStrain compare ISP1 ISP2 -o ISC
```

## Evaluation signals

- supplemental_table_s1.csv exists and contains per-tool shared-genome counts at each ANI threshold
- inStrain shared-genome counts agree with the values reported in the paper's Supplemental Table S1 and Figure 2d
- Each 'inStrain profile' output reports unmaskedBreadth, and shared-strain calls use the >= 0.5 unmaskedBreadth detection cutoff
- dRep dereplication was run at a 98% ANI threshold, matching the paper's recommendation to reduce multi-mapped reads
- Strain clustering used 'average' hierarchical clustering at 99.999% ANI

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 rather than included in the main text
- Genome de-replication above 98% ANI can cause multi-mapped reads (Supplemental Figure S4)
- Variant sites near the ends of genome fragments can be missed with inStrain (Supplemental Figure S3)
- inStrain's Python implementation is slow; the changelog notes it should call the node2vec C++ library

## Evidence

- [results] inStrain performs microdiversity-aware population comparisons that dramatically increase genomic comparison accuracy: [evidence span withheld — anchor status: not_found]
- [readme] Compare command: [evidence span withheld — anchor status: external_doc]
