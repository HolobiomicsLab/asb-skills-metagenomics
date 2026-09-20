---
name: strain-level-popani-comparison
description: Use when comparing microbial populations across metagenomic samples to
  quantify shared strains, needing pairwise popANI/ConANI and breadth-based detection
  against a concatenated reference genome set.
when_to_use_negative:
- The goal is rapid de-replication of a large genome set by average nucleotide identity
  rather than read-mapped, microdiversity-aware strain comparison (use dRep dereplicate
  instead)
- Reads are single-end or not properly paired, since inStrain removes all un-paired
  reads by default and filters read pairs in combination
- Genomes in the reference set are highly similar (e.g. not de-replicated around 98%
  ANI), which increases multi-mapped reads and distorts comparison accuracy
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: inStrain
  role: 'Core tool: runs ''inStrain profile'' on filtered .bam files to build IS_profile
    objects and ''inStrain compare'' to compute pairwise popANI/ConANI between samples'
  repo: github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps paired-end metagenomic reads to the concatenated reference genome .fasta
    with default settings (increase -X up to 1000 for longer inserts)
- name: Samtools
  role: Used by inStrain for interacting with .bam and .sam files and calculating
    per-position coverage
- name: dRep
  role: Baseline strain-comparison method benchmarked against inStrain; also recommended
    for de-replicating the reference genome set at ~98% ANI to reduce multi-mapped
    reads
  repo: github.com/MrOlm/drep
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:50dc2061d40548d9be38947690ab2ab32e9228b3de8f93a266a81bbe6973b32e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bowtie2
  - MIDAS
  - SNP Mutator
  - Samtools
  - dRep
  - inStrain
  - pIRS
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# strain-level-popani-comparison

## Summary

Computes microdiversity-aware, strain-level genomic comparisons between metagenomic samples using inStrain's profile-then-compare workflow: map reads with Bowtie2, profile each .bam, then run inStrain compare to obtain pairwise popANI and coverage overlap that are more accurate than dRep/MIDAS/StrainPhlAn ANI estimates.

## When to use

Use when comparing microbial populations across metagenomic samples to quantify shared strains, needing pairwise popANI/ConANI and breadth-based detection against a concatenated reference genome set.

## When NOT to use

- The goal is rapid de-replication of a large genome set by average nucleotide identity rather than read-mapped, microdiversity-aware strain comparison (use dRep dereplicate instead)
- Reads are single-end or not properly paired, since inStrain removes all un-paired reads by default and filters read pairs in combination
- Genomes in the reference set are highly similar (e.g. not de-replicated around 98% ANI), which increases multi-mapped reads and distorts comparison accuracy

## Inputs

- Paired-end Illumina metagenomic reads (e.g. ZymoBIOMICS mock community reads)
- Reference genome set in .fasta format (e.g. concatenated ZymoBIOMICS.STD.refseq.v2 genomes or E. coli SQ88 GCF_000988385.1)
- Sorted .bam files from Bowtie2 mapping, one per sample
- inStrain profile objects (IS_profile output directories) from inStrain profile

## Outputs

- Pairwise popANI and ConANI values for every pair of samples (inStrain compare output table)
- Coverage overlap metrics between sample pairs
- Detection calls per reference genome per sample (genome present with >= 0.5 unmaskedBreadth)
- Benchmark CSV/table comparing tool-calculated ANI vs true ANI (Supplemental Table S1 format)

## How to apply

Map each sample's paired-end reads to the reference genome set (e.g. the concatenated ZymoBIOMICS.STD.refseq.v2 .fasta) with Bowtie2 using default settings, raising the -X flag up to 1000 for longer-insert datasets. Run 'inStrain profile' on each .bam against the reference .fasta applying inStrain's default read filters: remove unpaired reads, require proper-pair orientation within expected insert size, a minimum mapQ, and a minimum read-pair nucleotide identity of 0.95. Then run 'inStrain compare' across all resulting IS_profile objects to compute pairwise popANI, ConANI, and coverage overlap between every pair of samples. Optionally define detection of each reference genome as >= 0.5 unmaskedBreadth when deciding which strains are present per sample. Rationale: inStrain accounts for within-population nucleotide diversity when scoring comparisons, which the paper shows dramatically increases genomic comparison accuracy relative to dRep, MIDAS, and StrainPhlAn benchmarked on synthetic mutated E. coli genomes (GCF_000988385.1) and ZymoBIOMICS mock community data.

## Related tools

- **inStrain** (Core tool: runs 'inStrain profile' on filtered .bam files to build IS_profile objects and 'inStrain compare' to compute pairwise popANI/ConANI between samples) — github.com/MrOlm/instrain
- **Bowtie2** (Maps paired-end metagenomic reads to the concatenated reference genome .fasta with default settings (increase -X up to 1000 for longer inserts))
- **Samtools** (Used by inStrain for interacting with .bam and .sam files and calculating per-position coverage)
- **dRep** (Baseline strain-comparison method benchmarked against inStrain; also recommended for de-replicating the reference genome set at ~98% ANI to reduce multi-mapped reads) — github.com/MrOlm/drep

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1; inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Reproduce the tool-accuracy comparison of Figure 2c: inStrain's popANI values closely match true ANI for closely related genomes while deviating for distantly related genomes, outperforming dRep, MIDAS, and StrainPhlAn
- Raw per-comparison values are recorded in a table matching Supplemental Table S1 format (tool, true ANI, calculated popANI) and exist as CSV
- Every IS_profile was generated with default read-pair ANI filter of 0.95 and paired-read/proper-orientation/insert-size filters applied
- Detection calls agree with the >= 0.5 unmaskedBreadth threshold per reference genome per sample
- Coverage and nucleotide diversity per genome, scaffold, and gene are reported for each profiled sample

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 (not included in the main text)
- Variant sites near the ends of genome fragments can be missed with inStrain (Supplemental Figure S3)
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads (Supplemental Figure S4)

## Evidence
