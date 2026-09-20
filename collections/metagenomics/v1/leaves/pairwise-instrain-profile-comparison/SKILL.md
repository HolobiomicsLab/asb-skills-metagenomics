---
name: pairwise-instrain-profile-comparison
description: Use when pairwise genomic comparison of the same genome set across metagenomic
  samples must distinguish true shared strains from co-occurring but divergent populations,
  e.g. twin strain-sharing benchmarks.
when_to_use_negative:
- When only genome-level dereplication of a single genome set is needed (no per-sample
  profiles) — use 'dRep dereplicate' instead
- When reads have not been mapped to a shared reference .fasta, since inStrain compare
  requires profile objects created from mappings to the same genome set
- When quick coverage/breadth calculation alone is sufficient — a tool like CoverM
  can compute coverage without microdiversity-aware comparison
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Profiles each sample's .bam ('inStrain profile') and performs the pairwise
    microdiversity-aware comparison ('inStrain compare') producing popANI and coverage
    overlap
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps each sample's reads to the concatenated sub-species genome .fasta with
    default settings to create the .bam inputs for profiling
- name: dRep
  role: 'Benchmark comparator: dereplicates the same genome set at 98% ANI to count
    genomes shared between infants by ANI-based criterion'
  repo: https://github.com/MrOlm/drep
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files during profiling
    and comparison
- name: Pandas
  role: Tabulates per-tool shared-genome counts for the supplemental comparison table
- name: Matplotlib
  role: Renders the comparison of shared-genome counts across tools (Figure 2d style)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:33724898a43fa274fcdc7b12b71ef2fc9ae1897c9323b8bfc45a611c7bfbfbdb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Bowtie2
  - Matplotlib
  - Pandas
  - Samtools
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
yes_rate: 0.5555555555555556
n_claims: 9
---

# pairwise-instrain-profile-comparison

## Summary

Compare inStrain profile objects built from different samples' read mappings to compute microdiversity-aware popANI and coverage overlap between all sample pairs, enabling accurate strain-sharing detection that outperforms ANI-only tools like dRep and MIDAS.

## When to use

Use when pairwise genomic comparison of the same genome set across metagenomic samples must distinguish true shared strains from co-occurring but divergent populations, e.g. twin strain-sharing benchmarks.

## When NOT to use

- When only genome-level dereplication of a single genome set is needed (no per-sample profiles) — use 'dRep dereplicate' instead
- When reads have not been mapped to a shared reference .fasta, since inStrain compare requires profile objects created from mappings to the same genome set
- When quick coverage/breadth calculation alone is sufficient — a tool like CoverM can compute coverage without microdiversity-aware comparison

## Inputs

- Bowtie2 .bam files from mapping each sample's paired reads to a concatenated sub-species genome .fasta
- Concatenated sub-species reference genome .fasta
- inStrain profile output directories (one per sample)
- Genome bins for comparison (e.g. infant genome bins)

## Outputs

- inStrain compare output with pairwise popANI, ConANI, and coverage-overlap values for all sample pairs
- Per-tool shared-genome counts at each ANI threshold (e.g. supplemental_table_s1.csv)
- Strain clusters defined at 99.999% ANI by average hierarchical clustering
- Comparison figure (e.g. Figure 2d style plot)

## How to apply

Map each sample's reads with Bowtie 2 (default settings) to the same concatenated sub-species genome .fasta and run 'inStrain profile' on each .bam, applying inStrain's read filtering (no unpaired reads, proper orientation/insert size, minimum mapQ and nucleotide identity). Then run 'inStrain compare' on all pairs of profile directories to obtain ConANI and popANI plus coverage-overlap metrics. Call a sub-species genome detected in a sample when it is present with >= 0.5 unmaskedBreadth, and cluster sub-species into strains using 'average' hierarchical clustering at a 99.999% ANI threshold. Benchmark the resulting shared-strain counts against dRep dereplication at 98% ANI (the recommended threshold to reduce multi-mapped reads), MIDAS, and StrainPhlAn run on the same read sets, tabulating shared genomes at each ANI threshold.

## Related tools

- **inStrain** (Profiles each sample's .bam ('inStrain profile') and performs the pairwise microdiversity-aware comparison ('inStrain compare') producing popANI and coverage overlap) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps each sample's reads to the concatenated sub-species genome .fasta with default settings to create the .bam inputs for profiling)
- **dRep** (Benchmark comparator: dereplicates the same genome set at 98% ANI to count genomes shared between infants by ANI-based criterion) — https://github.com/MrOlm/drep
- **Samtools** (Required by inStrain for interacting with .bam and .sam files during profiling and comparison)
- **Pandas** (Tabulates per-tool shared-genome counts for the supplemental comparison table)
- **Matplotlib** (Renders the comparison of shared-genome counts across tools (Figure 2d style))

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1; inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- supplemental_table_s1.csv exists and contains per-tool shared-genome counts
- inStrain shared-genome counts agree with the values reported in the paper's Supplemental Table S1 and Figure 2d
- Each sample has a completed inStrain profile directory containing popANI/comparison output from 'inStrain compare' run pairwise on all sample pairs
- Strain detection uses the defined criterion: sub-species present with >= 0.5 unmaskedBreadth in both infants, clustered at 99.999% ANI with average linkage
- dRep comparison uses a 98% ANI dereplication threshold consistent with reducing multi-mapped reads

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is recommended to reduce multi-mapped reads; skipping it can inflate false sharing
- The Python implementation (e.g. node2vec step) is slow; a C++ library would be needed for performance
- Raw benchmarking values are only available in Supplemental Table S1, not the main text

## Evidence

- [readme] README quick start: [evidence span withheld — anchor status: external_doc]
