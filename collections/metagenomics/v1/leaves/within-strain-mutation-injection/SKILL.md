---
name: within-strain-mutation-injection
description: Use when you have simulated longitudinal metagenomic reads (e.g. CAMISIM
  output directory or ART-simulated FASTQ reads) and need ground-truth within-strain
  SNV/InDel mutations injected across time points.
when_to_use_negative:
- The data are real metagenomic reads rather than simulated longitudinal/concurrent
  reads — STEMSIM only works well on species with high depth and well-characterized
  genomes
- The input is not simulated raw sequencing data and there is no CAMISIM output directory
  or externally simulated read set to mutate
- The goal is community-level abundance simulation rather than within-strain mutation
  injection — use CAMISIM alone for that purpose
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: STEMSim
  role: Core simulator that injects SNV and small InDel mutations into longitudinal
    simulated raw reads and outputs modified reads plus a VCF of true mutations
  repo: https://github.com/BoyanZhou/STEMSim
- name: CAMISIM
  role: Optional upstream generator of simulated microbial community reads used as
    input for camisim mode
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: Bowtie2
  role: Aligns longitudinal reads to each reference genome internally before mutation
    sampling
- name: samtools
  role: Processes Bowtie2 alignments during the mutation-injection workflow (tested
    with 1.9)
- name: ART
  role: Optional generator of simulated raw reads from a single reference genome for
    reads mode
- name: SnpEff
  role: Optionally annotates the VCF output of true mutations
- name: pysam
  role: Python library used by STEMSim to process alignment files during mutation
    injection (tested with 0.16.0.1)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:d7b0fbf8a35942eca3e24b03ce2ce0a698ec7db28efca41000d5bc326db7fcf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - ART
  - Bowtie2
  - CAMISIM
  - Python
  - STEMSim
  - SnpEff
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# within-strain-mutation-injection

## Summary

Inject SNV and small InDel mutations simulating within-strain short-term evolution into longitudinal simulated metagenomic reads using STEMSIM, producing modified read sets and a VCF of true injected mutations for benchmarking mutation-detection tools.

## When to use

Use when you have simulated longitudinal metagenomic reads (e.g. CAMISIM output directory or ART-simulated FASTQ reads) and need ground-truth within-strain SNV/InDel mutations injected across time points.

## When NOT to use

- The data are real metagenomic reads rather than simulated longitudinal/concurrent reads — STEMSIM only works well on species with high depth and well-characterized genomes
- The input is not simulated raw sequencing data and there is no CAMISIM output directory or externally simulated read set to mutate
- The goal is community-level abundance simulation rather than within-strain mutation injection — use CAMISIM alone for that purpose

## Inputs

- CAMISIM output directory (simulated longitudinal raw reads of a microbial community) for camisim mode
- genome_to_id.tsv mapping each genome_id to an absolute FASTA path (Tab-separated)
- STEMSim config.txt parameter file (template provided in repo)
- Subject ID for the longitudinal sample series
- For reads mode: raw read FASTQ files from longitudinal samples (paired files comma-separated, samples colon-separated), reference genome FASTA, and genome ID
- Optional: non-mutated region files (.gtf, .gff, or .bed) listed per genome ID
- Optional: SnpEff 5.1 directory and genomeID-to-snpEff ID matching table

## Outputs

- Modified raw metagenomics read sets carrying injected within-strain mutations
- VCF file recording the injected true SNV/InDel mutations (optionally annotated with SnpEff)
- STEMSim run log file

## How to apply

Run STEMSim (Python 3.6.5 with numpy 1.19.5 and pysam 0.16.0.1) in camisim mode by pointing --input_from_camisim at a CAMISIM output directory and --genome_to_id at the genome_to_id.tsv table mapping each genome_id to an absolute FASTA path, supplying the config.txt parameter file, a subject ID (-s), an output directory (-o), and a log path (-l). STEMSim internally aligns the longitudinal reads to each reference genome with Bowtie2, processes the alignments with samtools/pysam, and samples SNV and small InDel mutations across time points per genome while respecting any regions supplied via --nonmutated_list (.gtf, .gff, or .bed). Read sequences overlapping mutation sites are rewritten to carry the injected alleles, and the modified read sets are written to the output directory together with a run log. For reads simulated by other tools (e.g. ART) from a single reference genome, use reads mode with --input_reads (samples separated by ':', paired files by ','), --reference_genome, and --genome_id. Optionally annotate the VCF of true mutations with SnpEff 5.1 by supplying -p (snpEff directory) and -a (genomeID-to-snpEff ID matching table).

## Related tools

- **STEMSim** (Core simulator that injects SNV and small InDel mutations into longitudinal simulated raw reads and outputs modified reads plus a VCF of true mutations) — https://github.com/BoyanZhou/STEMSim
- **CAMISIM** (Optional upstream generator of simulated microbial community reads used as input for camisim mode) — https://github.com/CAMI-challenge/CAMISIM
- **Bowtie2** (Aligns longitudinal reads to each reference genome internally before mutation sampling)
- **samtools** (Processes Bowtie2 alignments during the mutation-injection workflow (tested with 1.9))
- **ART** (Optional generator of simulated raw reads from a single reference genome for reads mode)
- **SnpEff** (Optionally annotates the VCF output of true mutations)
- **pysam** (Python library used by STEMSim to process alignment files during mutation injection (tested with 0.16.0.1))

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m camisim -f PATH/config.txt -s test_subject1 --input_from_camisim PATH_to_output_directory_of_CAMISIM/ --genome_to_id PATH_to_genome_to_id/genome_to_id.tsv -o PATH_to_output_directory_of_STEMSim/ -l Path/stemsim_test1_log
```

## Evaluation signals

- The run completes without errors and the log file (-l) records the full STEMSim execution
- The output directory contains modified read files for each longitudinal time point
- The VCF of injected mutations lists SNV/InDel positions consistent with the mutation parameters configured in config.txt for each longitudinal sample
- Reads overlapping mutation sites carry the injected alleles when compared against the original CAMISIM/ART reads
- Genomic regions supplied via --nonmutated_list show no injected mutations

## Limitations

- STEMSim is designed for longitudinal or concurrent simulated raw sequencing reads; although it can run on real metagenomic data, it only works well on species with high depth and well-characterized genomes
- CAMISIM is only necessary for camisim mode; the reads mode handles raw reads from one reference genome, so multi-genome community input requires camisim mode
- SnpEff annotation of the VCF requires supplying both the snpEff directory (-p) and an ID-matching table (-a)

## Evidence

- [abstract] STEMSim outputs modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: [evidence span withheld — anchor status: not_found]
- [methods] stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data: "stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data"
- [readme] --nonmutated_list ... no mutation will be generated in the genomic regions listed in the genomic-region file: "Three formats (.gtf, .gff, and .bed) of genomic-region file are acceptable. For each genome listed, no mutation will be generated in the genomic regions listed in the genomic-region file"
- [readme] it only works well on species with high depth and well-characterized genomes: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes."
