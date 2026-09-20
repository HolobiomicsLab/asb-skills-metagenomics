---
name: cross-tool-ani-accuracy-comparison
description: Use when you need to quantitatively validate or compare ANI/strain-comparison
  accuracy of inStrain versus other tools (dRep, MIDAS, StrainPhlAn) using simulated
  genomes with known ground-truth ANI values.
when_to_use_negative:
- Input is real (non-simulated) metagenomic data where true ANI between strains is
  unknown — this skill depends on ground-truth ANI from SNP Mutator-mutated genomes
- Goal is ordinary genome dereplication or clustering of a genome set (use dRep dereplicate
  directly, e.g. dereplicating at 98% ANI to reduce multi-mapped reads) rather than
  tool-accuracy benchmarking
- Genomes are so distantly related that inStrain popANI is expected to deviate from
  true ANI — the reported Figure 2b trend holds for closely related genomes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: inStrain
  role: Profiles each .bam ('inStrain profile') and computes popANI between mutated
    and original genome via 'inStrain compare'; the tool being benchmarked
  repo: github.com/MrOlm/instrain
- name: SNP Mutator
  role: Introduces a known number of mutations into the SQ88 reference genome to create
    genomes with known true ANI
- name: pIRS
  role: Simulates Illumina reads at 20x coverage from each mutated genome
- name: Bowtie2
  role: Maps simulated reads back to the original reference genome to produce .bam
    files for profiling
- name: dRep
  role: Baseline strain-comparison tool run on the same simulated read sets to obtain
    competing ANI estimates
  repo: github.com/MrOlm/drep
- name: Samtools
  role: Handles .bam/.sam files underlying inStrain's read filtering and coverage
    calculation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:d14e4fc2a74f37533c00195a13d65870fb231d528ee08d9576e761b58e21e000
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-tool-ani-accuracy-comparison

## Summary

Benchmark the accuracy of strain-level ANI estimation across tools by simulating genomes of known true ANI (SNP Mutator mutations of an E. coli SQ88 reference), generating synthetic Illumina reads at 20x coverage with pIRS, and comparing inStrain's microdiversity-aware popANI against dRep, MIDAS, and StrainPhlAn estimates. This reproduces the paper's Figure 2b / Supplemental Table S1 validation showing inStrain's popANI closely tracks true ANI for closely related genomes.

## When to use

Use when you need to quantitatively validate or compare ANI/strain-comparison accuracy of inStrain versus other tools (dRep, MIDAS, StrainPhlAn) using simulated genomes with known ground-truth ANI values.

## When NOT to use

- Input is real (non-simulated) metagenomic data where true ANI between strains is unknown — this skill depends on ground-truth ANI from SNP Mutator-mutated genomes
- Goal is ordinary genome dereplication or clustering of a genome set (use dRep dereplicate directly, e.g. dereplicating at 98% ANI to reduce multi-mapped reads) rather than tool-accuracy benchmarking
- Genomes are so distantly related that inStrain popANI is expected to deviate from true ANI — the reported Figure 2b trend holds for closely related genomes

## Inputs

- E. coli strain SQ88 reference genome FASTA (RefSeq GCF_000988385.1)
- Series of SNP-mutated genome FASTAs with known true ANI values relative to the reference
- Simulated Illumina paired-end reads (pIRS, 20x coverage) per mutated genome
- Bowtie2 .bam files mapping simulated reads back to the original reference genome
- inStrain IS_profile output objects from 'inStrain profile'

## Outputs

- Benchmark table (CSV) with columns for tool, true ANI, and calculated popANI (formatted as Figure 2b / Supplemental Table S1)
- Pairwise popANI values from 'inStrain compare' across all mutated-genome IS_profile pairs
- ANI estimates from baseline tools (dRep, MIDAS, StrainPhlAn) on the same simulated read sets

## How to apply

Download the E. coli strain SQ88 reference genome (RefSeq accession GCF_000988385.1) and use SNP Mutator to introduce a known number of mutations, producing mutated genomes whose true ANI versus the reference is known. Simulate Illumina reads from each mutated genome at 20x coverage with pIRS, then map the synthetic reads back to the original reference with Bowtie2. Run 'inStrain profile' on each resulting .bam with the default minimum read-pair ANI of 0.95, then run 'inStrain compare' across the IS_profile objects to obtain pairwise popANI values — inStrain performs strain-level comparisons by comparing inStrain profile objects created by mapping different sets of reads. Run the baseline strain-comparison tools (dRep, MIDAS, StrainPhlAn) on the same simulated read sets to obtain their ANI estimates, then compile calculated popANI versus true ANI for every tool into a benchmark table formatted as Figure 2b / Supplemental Table S1.

## Related tools

- **inStrain** (Profiles each .bam ('inStrain profile') and computes popANI between mutated and original genome via 'inStrain compare'; the tool being benchmarked) — github.com/MrOlm/instrain
- **SNP Mutator** (Introduces a known number of mutations into the SQ88 reference genome to create genomes with known true ANI)
- **pIRS** (Simulates Illumina reads at 20x coverage from each mutated genome)
- **Bowtie2** (Maps simulated reads back to the original reference genome to produce .bam files for profiling)
- **dRep** (Baseline strain-comparison tool run on the same simulated read sets to obtain competing ANI estimates) — github.com/MrOlm/drep
- **Samtools** (Handles .bam/.sam files underlying inStrain's read filtering and coverage calculation)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1; inStrain compare inStrain_profile1 inStrain_profile2
```

## Evaluation signals

- Output table exists in CSV format with columns for tool, true ANI, and calculated popANI
- inStrain popANI values closely match true ANI for closely related genomes (near-identical mutated genomes)
- popANI deviates from true ANI for distantly related genomes, consistent with the reported Figure 2b trend
- Each mutated genome's true ANI is derivable from the known number of SNP Mutator-introduced mutations divided by genome length
- Every tool was run on the same simulated read sets (20x pIRS, Bowtie2-mapped), making the comparison like-for-like

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 and are not reproduced in the main text
- inStrain popANI closely matches true ANI only for closely related genomes and deviates for distantly related ones
- Genome de-replication at 98% ANI is recommended (e.g. via dRep) to reduce multi-mapped reads, which affects comparison accuracy

## Evidence

- [discussion] SNP Mutator mutates reference to known ANI: "were performed by using SNP Mutator 60 to introduce a known number of mutations into a reference genome"
- [discussion] pIRS simulation at 20x: "Illumina reads were simulated for all genomes at 20x coverage using pIRS 61"
- [discussion] Reference genome used for benchmark: "reference genome (Escherichia coli strain SQ88; RefSeq accession number GCF_000988385.1)"
- [discussion] How inStrain compares profiles: "InStrain performs strain-level comparisons by comparing inStrain profile objects that were created by mapping different sets of reads"
- [readme] compare command: [evidence span withheld — anchor status: external_doc]
