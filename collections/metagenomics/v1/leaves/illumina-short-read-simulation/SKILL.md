---
name: illumina-short-read-simulation
description: Use when you need a known-composition paired-end Illumina FASTQ set from
  reference bacterial assemblies to quantify what fraction of true bacterial reads
  a decontamination tool retains or incorrectly removes.
when_to_use_negative:
- Input is real metagenomic or clinical data with unknown composition — simulation
  is for ground-truth benchmarking, not production decontamination
- You need long-read (ONT) benchmarks — this skill covers paired short reads; use
  PBSIM2-based ONT simulation instead
- The goal is to measure human read removal efficiency — use real human reads (e.g.
  1000 Genomes Project samples) rather than simulated bacterial reads
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: DWGSIM
  role: Simulates paired Illumina short reads from each complete bacterial assembly
    to create the benchmark read set
- name: Hostile
  role: Decontaminates the simulated reads in Bowtie2 mode against the default human-t2t-hla
    index; retention is computed from its JSON logs
  repo: github.com/bede/hostile
- name: Bowtie2
  role: Alignment engine used automatically by Hostile for short reads with --sensitive
    parameters and -k 1
- name: HRRT
  role: Comparison decontamination tool (sra-human-scrubber) run on the same simulated
    reads to quantify relative bacterial read loss
  repo: ncbi/sra-human-scrubber
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:7135cc8809d907ee89f1e72fa0cc9a8a0fe3bf723a3b98f139cb51a24967adb7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie2
  - DWGSIM
  - HRRT
  - Hostile
  - PBSIM2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# illumina-short-read-simulation

## Summary

Simulate paired Illumina short reads from complete bacterial genome assemblies (e.g. 985 FDA-ARGOS or 140 mycobacterial genomes) using DWGSIM to create a ground-truth read set for benchmarking host-decontamination tools such as Hostile and HRRT.

## When to use

Use when you need a known-composition paired-end Illumina FASTQ set from reference bacterial assemblies to quantify what fraction of true bacterial reads a decontamination tool retains or incorrectly removes.

## When NOT to use

- Input is real metagenomic or clinical data with unknown composition — simulation is for ground-truth benchmarking, not production decontamination
- You need long-read (ONT) benchmarks — this skill covers paired short reads; use PBSIM2-based ONT simulation instead
- The goal is to measure human read removal efficiency — use real human reads (e.g. 1000 Genomes Project samples) rather than simulated bacterial reads

## Inputs

- Complete bacterial genome assemblies in FASTA (e.g. 985 FDA-ARGOS reference grade genomes, BioProject 231221; or 140 complete mycobacterial assemblies)
- Simulated paired Illumina FASTQ files (R1/R2) produced by DWGSIM

## Outputs

- DWGSIM-simulated paired Illumina FASTQ files per assembly
- Hostile JSON clean logs (reads_in, reads_out, reads_removed, reads_removed_proportion) and cleaned FASTQs
- Retention comparison table, e.g. bacterial_retention_comparison.csv (Hostile 99.99989% vs HRRT 99.99752% retained on FDA-ARGOS Illumina data)

## How to apply

Download the complete, reference-grade bacterial assemblies (e.g. the 985 FDA-ARGOS genomes from BioProject 231221, or 140 complete mycobacterial assemblies) and run DWGSIM on each to produce paired Illumina short reads. Feed the simulated paired FASTQs into hostile clean in its default short-read mode, where Bowtie2 is used automatically with --sensitive parameters (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15) and -k 1 against the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51), capturing the JSON log. Because every read is known to be bacterial, the retention ratio reads_out/reads_in (from the reads_in, reads_out, reads_removed, reads_removed_proportion fields) directly measures false removal without requiring taxonomic labels. Compare against a competing tool such as HRRT run on the same simulated reads, and tabulate retention values into a comparison table (e.g. bacterial_retention_comparison.csv).

## Related tools

- **DWGSIM** (Simulates paired Illumina short reads from each complete bacterial assembly to create the benchmark read set)
- **Hostile** (Decontaminates the simulated reads in Bowtie2 mode against the default human-t2t-hla index; retention is computed from its JSON logs) — github.com/bede/hostile
- **Bowtie2** (Alignment engine used automatically by Hostile for short reads with --sensitive parameters and -k 1)
- **HRRT** (Comparison decontamination tool (sra-human-scrubber) run on the same simulated reads to quantify relative bacterial read loss) — ncbi/sra-human-scrubber

## Examples

```
hostile clean --fastq1 sim_R1.fastq.gz --fastq2 sim_R2.fastq.gz --index human-t2t-hla
```

## Evaluation signals

- Hostile retention on simulated FDA-ARGOS Illumina reads must reach ≥99.989% (reported 99.99989%), satisfying the abstract-level guarantee
- Hostile retention should exceed HRRT retention on the same simulated read set (HRRT reported at 99.99752%, removing a mean of 21x, 95% CI: 9–30, more bacterial reads)
- For mycobacterial Illumina simulations, Hostile retention must reproduce ~99.99998% vs HRRT ~99.999751%
- Retention ratios must be derivable from Hostile JSON logs as reads_out/reads_in, with reads_removed_proportion consistent with the reported table
- Masked index control: switching to human-t2t-hla-argos985 should raise retention to ≥99.997% with ≤0.001% reduction in human read removal

## Limitations

- Retention figures are simulation-dependent; DWGSIM's error model may not fully capture real Illumina error profiles, so absolute values may not transfer to real data
- Results are specific to the human-t2t-hla default index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51); other indexes yield different retention
- Hostile requires ~4GB RAM for short-read (Bowtie2) decontamination and earlier versions could corrupt empty fastq.gz files when no reads remain after decontamination

## Evidence

- [methods] 985 reference grade bacterial genomes](https://www.ncbi.nlm.nih.gov/bioproject/231221) including common human pathogens: "985 reference grade bacterial genomes](https://www.ncbi.nlm.nih.gov/bioproject/231221) including common human pathogens"
- [abstract] our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [abstract] Using Hostile with a masked reference genome further increases bacterial read retention (≥99.997%) with negligible (≤0.001%) reduction in human read removal performance: "Using Hostile with a masked reference genome further increases bacterial read retention (≥99.997%) with negligible (≤0.001%) reduction in human read removal performance"
- [other] hostile clean --fastq1 <r1> --fastq2 <r2>: [evidence span withheld — anchor status: not_found]
- [readme] The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
