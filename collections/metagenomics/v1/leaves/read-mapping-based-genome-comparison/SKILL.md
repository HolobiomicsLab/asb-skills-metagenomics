---
name: read-mapping-based-genome-comparison
description: Use when you need accurate strain-level genome similarity (ANI) between
  genomes across metagenomic samples — e.g. benchmarking genome comparison accuracy
  or detecting shared strains — from read mappings rather than raw assemblies.
when_to_use_negative:
- When genomes are only available as assemblies and no read mappings exist — assembly-based
  tools like dRep's built-in ANI operate directly on FASTA files
- When you only need coverage/breadth quantification of genomes rather than strain-level
  similarity — CoverM or 'inStrain profile' alone suffices
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: inStrain
  role: Profiles each .bam mapping ('inStrain profile') and performs pairwise strain-level
    genome comparison ('inStrain compare') yielding ConANI and popANI
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps simulated Illumina reads to the reference genome(s) with default settings
    to produce the .bam inputs
- name: pIRS
  role: Simulates Illumina reads from mutated genomes at 20x coverage for benchmarking
- name: SNP Mutator
  role: Introduces a known number of mutations into the reference genome to generate
    genomes with known true ANI values
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files and calculating
    coverage per position
- name: dRep
  role: Baseline strain-comparison tool whose ANI estimates are benchmarked against
    inStrain popANI
  repo: https://github.com/MrOlm/drep
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:6796c2ea56d7ebbc0bf9e6406d44d6a2b28fe58f9b4017e6b419a6597aa973e8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Bowtie2
  - SNP Mutator
  - Samtools
  - dRep
  - inStrain
  - pIRS
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# read-mapping-based-genome-comparison

## Summary

Compare microbial genomes by mapping reads to reference genomes, profiling each mapping with inStrain, and then running 'inStrain compare' to obtain microdiversity-aware population-level ANI (popANI) and consensus ANI (ConANI) values that are more accurate than strain-comparison methods operating on assemblies alone.

## When to use

Use when you need accurate strain-level genome similarity (ANI) between genomes across metagenomic samples — e.g. benchmarking genome comparison accuracy or detecting shared strains — from read mappings rather than raw assemblies.

## When NOT to use

- When genomes are only available as assemblies and no read mappings exist — assembly-based tools like dRep's built-in ANI operate directly on FASTA files
- When you only need coverage/breadth quantification of genomes rather than strain-level similarity — CoverM or 'inStrain profile' alone suffices

## Inputs

- Sorted .bam files from Bowtie2 mapping of Illumina reads to reference genome FASTA(s)
- Reference nucleotide FASTA (e.g. E. coli SQ88 GCF_000988385.1)
- inStrain profile output objects (IS_profile directories) from 'inStrain profile'

## Outputs

- IS_compare object with pairwise ConANI and popANI values for each genome pair
- Benchmark CSV table with columns for tool, true ANI, and calculated popANI (as in Figure 2b / Supplemental Table S1)

## How to apply

Simulate or obtain Illumina reads at ~20x coverage per genome, map them to the reference genome(s) with Bowtie2 (using default settings as in the benchmark), and run 'inStrain profile' on each .bam file with the default minimum read-pair ANI of 0.95 to produce IS_profile objects. Then run 'inStrain compare' across the resulting IS_profile objects, which computes pairwise ConANI and microdiversity-aware popANI values by comparing positions covered in both profiles. For validation, mutate a reference genome (e.g. E. coli strain SQ88, RefSeq GCF_000988385.1) with SNP Mutator to known true ANI values, simulate reads with pIRS, and confirm popANI closely matches true ANI for closely related genomes while deviating for distantly related ones. inStrain's careful .bam filtering (removing unpaired reads, requiring proper pair orientation within expected insert size, minimum mapQ, and minimum nucleotide identity) is what reduces erroneously mapped reads and drives the accuracy gain over dRep, MIDAS, and StrainPhlAn.

## Related tools

- **inStrain** (Profiles each .bam mapping ('inStrain profile') and performs pairwise strain-level genome comparison ('inStrain compare') yielding ConANI and popANI) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps simulated Illumina reads to the reference genome(s) with default settings to produce the .bam inputs)
- **pIRS** (Simulates Illumina reads from mutated genomes at 20x coverage for benchmarking)
- **SNP Mutator** (Introduces a known number of mutations into the reference genome to generate genomes with known true ANI values)
- **Samtools** (Required by inStrain for interacting with .bam and .sam files and calculating coverage per position)
- **dRep** (Baseline strain-comparison tool whose ANI estimates are benchmarked against inStrain popANI) — https://github.com/MrOlm/drep

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1; inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Output comparison table exists in CSV format with columns for tool, true ANI, and calculated popANI
- inStrain popANI values closely match true ANI for closely related genomes (known mutation counts from SNP Mutator)
- popANI deviates from true ANI for distantly related genomes, consistent with the Figure 2b trend
- 'inStrain compare' ran across all IS_profile objects and produced both ConANI and popANI for every genome pair

## Limitations

- popANI deviates from true ANI for distantly related genomes, so it is most trustworthy for close strain comparisons
- Raw benchmarking values are deferred to Supplemental Table S1 and are not included in the main text
- The comparison is only as good as the underlying mapping — reads erroneously mapped inflate similarity, hence the strict .bam filters (paired-only, proper orientation, min mapQ, min nucleotide identity)

## Evidence

- [readme] compare command usage: [evidence span withheld — anchor status: external_doc]
