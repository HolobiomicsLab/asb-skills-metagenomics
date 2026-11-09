---
name: genome-mutation-simulation-with-known-ani
description: Use when benchmarking ANI/strain-comparison tools (inStrain popANI, dRep,
  MIDAS, StrainPhlAn) on synthetic data where the ground-truth ANI between genomes
  is known by construction.
when_to_use_negative:
- Input is real metagenomic data rather than simulated reads — this skill's validation
  depends on known true ANI from controlled mutation counts
- The goal is ANI estimation between genuinely divergent genomes (>~95-99% ANI) where
  read-mapping-based popANI is expected to deviate from true ANI rather than match
  it
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SNP Mutator
  role: Introduces a known number of mutations into the reference genome to create
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
  role: Baseline strain-comparison tool whose ANI estimates are benchmarked against
    inStrain popANI
  repo: github.com/MrOlm/drep
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:d40eeedb40db6bc16655c71ce2258c126f03e426908c6b40fef62e1eefb1ab2f
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# genome-mutation-simulation-with-known-ani

## Summary

Benchmark protocol for strain-comparison accuracy: introduce a known number of mutations into a reference genome with SNP Mutator, simulate Illumina reads with pIRS at 20x coverage, map them with Bowtie2, and compare calculated popANI from inStrain against the true ANI.

## When to use

Use when benchmarking ANI/strain-comparison tools (inStrain popANI, dRep, MIDAS, StrainPhlAn) on synthetic data where the ground-truth ANI between genomes is known by construction.

## When NOT to use

- Input is real metagenomic data rather than simulated reads — this skill's validation depends on known true ANI from controlled mutation counts
- The goal is ANI estimation between genuinely divergent genomes (>~95-99% ANI) where read-mapping-based popANI is expected to deviate from true ANI rather than match it

## Inputs

- Reference genome FASTA (E. coli strain SQ88, RefSeq GCF_000988385.1)
- Mutated genome FASTA files with known numbers of introduced SNPs (SNP Mutator output)
- Simulated Illumina paired-end reads from pIRS at 20x coverage
- BAM files from Bowtie2 mapping of simulated reads to the original reference

## Outputs

- inStrain IS_profile objects per mutated genome ('inStrain profile' output)
- popANI values from 'inStrain compare' for each mutated genome vs. the original
- Benchmark CSV table with columns for tool, true ANI, and calculated popANI (Figure 2b / Supplemental Table S1 format)

## How to apply

Download the E. coli strain SQ88 reference genome (RefSeq GCF_000988385.1) and use SNP Mutator to introduce a known number of mutations, generating mutated genomes with known true ANI values. Simulate Illumina reads from each mutated genome with pIRS at 20x coverage and map reads back to the original reference using Bowtie2. Run 'inStrain profile' on each .bam with the default minimum read-pair ANI of 0.95, then run 'inStrain compare' across the resulting IS_profile objects to obtain popANI for each mutated genome versus the original. Run baseline tools (dRep, MIDAS, StrainPhlAn) on the same simulated read sets, then compile calculated popANI versus true ANI into a benchmark table (Figure 2b / Supplemental Table S1).

## Related tools

- **SNP Mutator** (Introduces a known number of mutations into the reference genome to create mutated genomes with known true ANI values)
- **pIRS** (Simulates Illumina reads from each mutated genome at 20x coverage)
- **Bowtie2** (Maps the simulated reads back to the original reference genome to produce .bam files for profiling)
- **inStrain** (Profiles each .bam ('inStrain profile') and computes popANI between profiles ('inStrain compare')) — github.com/MrOlm/instrain
- **Samtools** (Required by inStrain for interacting with .bam and .sam files)
- **dRep** (Baseline strain-comparison tool whose ANI estimates are benchmarked against inStrain popANI) — github.com/MrOlm/drep

## Examples

```
inStrain profile mutated_vs_ref.bam GCF_000988385.1.fasta -o IS_profile_mut1; inStrain compare IS_profile_mut1 IS_profile_ref -o IS_compare
```

## Evaluation signals

- Output benchmark table exists in CSV format with columns for tool, true ANI, and calculated popANI
- inStrain popANI values closely match true ANI for closely related (heavily mutated-but-similar) genomes
- popANI values deviate from true ANI for distantly related genomes, consistent with the Figure 2b trend
- Each simulated read set is at 20x coverage as specified by the pIRS simulation parameters
- 'inStrain compare' runs complete across all IS_profile objects, yielding pairwise popANI for every mutated genome vs. the original

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 and are not included in the main text
- inStrain's popANI is expected to deviate from true ANI for distantly related genomes, so the metric is most accurate for closely related strains
- The benchmark relies on a single reference genome (E. coli SQ88) and simulated Illumina data, so results may not transfer directly to other organisms or sequencing technologies

## Evidence

- [readme] README command for profiling a BAM: [evidence span withheld — anchor status: external_doc]
- [readme] README command for strain-level comparison: [evidence span withheld — anchor status: external_doc]
