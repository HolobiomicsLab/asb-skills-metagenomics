---
name: illumina-read-simulation-from-mutated-genomes
description: Use when benchmarking ANI/strain-comparison methods on metagenomic data
  and you need synthetic read sets with known ground-truth ANI values derived from
  genomes mutated at controlled rates.
when_to_use_negative:
- You need to compare real biological samples rather than validate method accuracy
  on simulated ground truth — use inStrain profile/compare directly on real metagenomic
  .bam files instead
- Your input is already a set of unmapped real reads with no known reference ANI;
  there is no ground truth to benchmark against
- You need long-read or non-Illumina read simulation — pIRS simulates Illumina reads
  only
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SNP Mutator
  role: Introduces a known number of mutations into the reference genome to generate
    mutated genomes with known true ANI values
- name: pIRS
  role: Simulates Illumina reads from each mutated genome at 20x coverage
- name: Bowtie2
  role: Maps the simulated reads back to the original reference genome to produce
    .bam files for profiling
- name: inStrain
  role: Profiles each .bam ('inStrain profile') and computes popANI between profiles
    ('inStrain compare')
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files
- name: dRep
  role: Baseline genome-comparison tool run on the same simulated read sets for benchmarking
    ANI estimates
  repo: github.com/MrOlm/drep
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:8a2823d26852803025d34b06ed70dd6e602d598cd438745bf37c63ceccfc442a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bowtie2
  - SNP Mutator
  - Samtools
  - dRep
  - inStrain
  - pIRS
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# illumina-read-simulation-from-mutated-genomes

## Summary

Benchmark strain-comparison accuracy by introducing a known number of mutations into a reference genome (E. coli strain SQ88, RefSeq GCF_000988385.1) with SNP Mutator, simulating 20x Illumina reads with pIRS, and mapping them back with Bowtie2 so that calculated popANI can be compared against true ANI.

## When to use

Use when benchmarking ANI/strain-comparison methods on metagenomic data and you need synthetic read sets with known ground-truth ANI values derived from genomes mutated at controlled rates.

## When NOT to use

- You need to compare real biological samples rather than validate method accuracy on simulated ground truth — use inStrain profile/compare directly on real metagenomic .bam files instead
- Your input is already a set of unmapped real reads with no known reference ANI; there is no ground truth to benchmark against
- You need long-read or non-Illumina read simulation — pIRS simulates Illumina reads only

## Inputs

- Reference genome FASTA (E. coli strain SQ88, RefSeq accession GCF_000988385.1)
- Known mutation counts per mutated genome (from SNP Mutator)
- Simulated Illumina paired-end reads at 20x coverage (pIRS output FASTQ)
- Bowtie2 .bam files of synthetic reads mapped to the original reference genome

## Outputs

- Mutated genome FASTA files with known true ANI values relative to the reference
- per-genome inStrain IS_profile objects
- Benchmark table (CSV, as in Figure 2b / Supplemental Table S1) with columns for tool, true ANI, and calculated popANI

## How to apply

Download the reference genome (Escherichia coli strain SQ88; RefSeq GCF_000988385.1) and use SNP Mutator to introduce a known number of mutations, generating mutated genomes whose true ANI to the original is known by construction. Simulate Illumina reads for all mutated genomes at 20x coverage using pIRS, then map the synthetic reads back to the original reference genome using Bowtie2. Run 'inStrain profile' on each resulting .bam file (default minimum read-pair ANI of 0.95), then 'inStrain compare' across the IS_profile objects to obtain popANI values for each mutated genome versus the original. Run baseline tools (dRep, MIDAS, StrainPhlAn) on the same simulated read sets and compile calculated popANI versus true ANI into a benchmark table formatted as in Figure 2b / Supplemental Table S1.

## Related tools

- **SNP Mutator** (Introduces a known number of mutations into the reference genome to generate mutated genomes with known true ANI values)
- **pIRS** (Simulates Illumina reads from each mutated genome at 20x coverage)
- **Bowtie2** (Maps the simulated reads back to the original reference genome to produce .bam files for profiling)
- **inStrain** (Profiles each .bam ('inStrain profile') and computes popANI between profiles ('inStrain compare')) — github.com/MrOlm/instrain
- **Samtools** (Required by inStrain for interacting with .bam and .sam files)
- **dRep** (Baseline genome-comparison tool run on the same simulated read sets for benchmarking ANI estimates) — github.com/MrOlm/drep

## Examples

```
$ inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
$ inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Output benchmark table exists in CSV format with columns for tool, true ANI, and calculated popANI
- inStrain popANI values closely match true ANI for closely related genomes
- inStrain popANI deviates from true ANI for distantly related genomes, consistent with the reported Figure 2b trend
- Each mutated genome has a known, explicitly counted number of mutations introduced by SNP Mutator, giving a verifiable true ANI
- Simulated read sets match the 20x coverage used in the published benchmark

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 rather than the main text
- Simulated reads are Illumina-only (pIRS) and do not capture real metagenomic complexity such as multi-strain communities or uneven coverage
- inStrain variant sites near the ends of genome fragments can be missed, which may affect SNV-based comparisons at fragment edges

## Evidence

- [readme] inStrain commands: [evidence span withheld — anchor status: not_found]
- [results] Validation trend: [evidence span withheld — anchor status: not_found]
