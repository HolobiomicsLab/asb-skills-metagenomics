---
name: longitudinal-sample-mutation-modeling
description: Use when evaluating tools that detect short-term within-strain evolutionary
  mutations in longitudinal metagenomic data, or when ground-truth mutated reads with
  a known mutation VCF are needed for simulated time-series communities.
when_to_use_negative:
- Input is real metagenomic data from species with low sequencing depth or poorly
  characterized genomes — the README warns STEMSim 'only works well on species with
  high depth and well-characterized genomes'
- Goal is to simulate the microbial community abundance and reads themselves — use
  CAMISIM (or ART for single-genome reads) upstream; STEMSim only injects mutations
  on already-simulated reads
- Data are not longitudinal/concurrent time-point samples — the README states STEMSim
  'is designed to work on longitudinal or concurrent simulated raw sequencing reads'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: STEMSim
  role: Core simulator that injects within-strain SNV/InDel mutations into longitudinal
    simulated reads and outputs the mutated reads plus a VCF of true mutations
  repo: https://github.com/BoyanZhou/STEMSim
- name: CAMISIM
  role: Optional upstream generator of the simulated microbial community longitudinal
    raw reads used as 'camisim'-mode input
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: Bowtie2
  role: Aligns the longitudinal reads to each reference genome internally before mutation
    sampling
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Processes the Bowtie2 alignments during STEMSim's internal pipeline
  repo: http://www.htslib.org/doc/
- name: ART
  role: Optional reads simulator generating the single-genome raw reads consumed by
    'reads' mode
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- name: SnpEff
  role: Optional annotation of the VCF output of injected true mutations
  repo: http://pcingola.github.io/SnpEff/
- name: pysam
  role: Python library (tested with 0.16.0.1) used by STEMSim for SAM/BAM alignment
    processing
- name: numpy
  role: Python dependency (tested with 1.19.5) supporting STEMSim's mutation sampling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:0f8afbe56e7eb78f652d44b27ac12bcf49f6eaf62cc2d7d637213a4e47ea2dda
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# longitudinal-sample-mutation-modeling

## Summary

Simulate within-strain short-term evolutionary mutations (SNVs and small InDels) on longitudinal metagenomic sequencing data using STEMSim, producing modified raw reads and a VCF of true injected mutations for benchmarking mutation-detection tools.

## When to use

Use when evaluating tools that detect short-term within-strain evolutionary mutations in longitudinal metagenomic data, or when ground-truth mutated reads with a known mutation VCF are needed for simulated time-series communities.

## When NOT to use

- Input is real metagenomic data from species with low sequencing depth or poorly characterized genomes — the README warns STEMSim 'only works well on species with high depth and well-characterized genomes'
- Goal is to simulate the microbial community abundance and reads themselves — use CAMISIM (or ART for single-genome reads) upstream; STEMSim only injects mutations on already-simulated reads
- Data are not longitudinal/concurrent time-point samples — the README states STEMSim 'is designed to work on longitudinal or concurrent simulated raw sequencing reads'

## Inputs

- CAMISIM output directory (simulated longitudinal raw shotgun metagenomic reads) for 'camisim' mode
- genome_to_id.tsv table mapping each genome_id to the absolute path of its FASTA reference
- STEMSim config.txt parameter file (template provided; parameter tutorial on the wiki)
- Subject ID for the longitudinal samples
- Optionally: raw FASTQ read files from other simulators (e.g. ART) for 'reads' mode, with a reference genome FASTA and genome_id
- Optionally: non-mutated genomic-region file list (.gtf, .gff, or .bed formats) via --nonmutated_list
- Optionally: SnpEff directory and genomeID-to-snpEff ID-matching table for VCF annotation

## Outputs

- Modified longitudinal raw sequencing read sets carrying the injected within-strain SNVs and small InDels
- VCF file recording the injected true mutations (optionally annotated with SnpEff 5.1)
- STEMSim run log file

## How to apply

Run STEMSim (Python 3.6.5, numpy 1.19.5, pysam 0.16.0.1) in 'camisim' mode, passing the CAMISIM output directory, the config.txt parameter file (template provided in the repository), the genome_to_id.tsv table mapping each genome_id to its absolute FASTA path, a subject ID (-s), an output directory (-o), and a log path (-l). For reads simulated outside CAMISIM (e.g. with ART), use 'reads' mode, supplying longitudinal FASTQ files separated by ':' (paired mates separated by ','), the reference genome FASTA, and a genome_id. STEMSim internally aligns the longitudinal reads to each reference genome with Bowtie2, processes alignments with samtools/pysam, and samples SNV and small InDel mutations across time points per genome while skipping genomic regions listed in an optional non-mutated region file (.gtf, .gff, or .bed via --nonmutated_list or --nonmutated_file). Read sequences overlapping mutation sites are rewritten to carry the injected alleles, and the injected true mutations are exported as a VCF that can optionally be annotated with SnpEff 5.1 using -p (snpEff directory) and -a (ID-matching table).

## Related tools

- **STEMSim** (Core simulator that injects within-strain SNV/InDel mutations into longitudinal simulated reads and outputs the mutated reads plus a VCF of true mutations) — https://github.com/BoyanZhou/STEMSim
- **CAMISIM** (Optional upstream generator of the simulated microbial community longitudinal raw reads used as 'camisim'-mode input) — https://github.com/CAMI-challenge/CAMISIM
- **Bowtie2** (Aligns the longitudinal reads to each reference genome internally before mutation sampling) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Processes the Bowtie2 alignments during STEMSim's internal pipeline) — http://www.htslib.org/doc/
- **ART** (Optional reads simulator generating the single-genome raw reads consumed by 'reads' mode) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- **SnpEff** (Optional annotation of the VCF output of injected true mutations) — http://pcingola.github.io/SnpEff/
- **pysam** (Python library (tested with 0.16.0.1) used by STEMSim for SAM/BAM alignment processing)
- **numpy** (Python dependency (tested with 1.19.5) supporting STEMSim's mutation sampling)

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m camisim -f PATH/config.txt -s test_subject1 --input_from_camisim PATH_to_output_directory_of_CAMISIM/ --genome_to_id PATH_to_genome_to_id/genome_to_id.tsv -o PATH_to_output_directory_of_STEMSim/ -l Path/stemsim_test1_log
```

## Evaluation signals

- The run completes without errors and the output directory contains the modified read files plus the VCF of injected mutations
- The mutation record VCF lists SNV/InDel positions consistent with the mutation parameters configured in config.txt for each longitudinal sample
- Reads overlapping injected mutation sites carry the injected alleles after STEMSim's sequence rewriting
- Genomes/regions listed in the non-mutated region file (.gtf/.gff/.bed) show no injected mutations
- In 'camisim' mode, the genome_to_id.tsv entries resolve to valid absolute FASTA paths for every genome_id in the CAMISIM output

## Limitations

- STEMSim is designed for simulated longitudinal or concurrent reads; although it can run on real metagenomic data, it only works well on species with high depth and well-characterized genomes
- CAMISIM is only necessary for 'camisim' mode; 'reads' mode handles reads from one reference genome at a time
- Tested with specific versions: Bowtie2 (0.16.0.1), samtools (1.9), CAMISIM (1.3), ART (2016-06-05), SnpEff (5.1), Python 3.6.5, numpy 1.19.5, pysam 0.16.0.1
- No changelog found for the BoyanZhou/STEMSim repository

## Evidence

- [abstract] we introduce STEMSIM, a user-friendly command-line simulator of short-term evolutionary mutations for longitudinal metagenomic data: "we introduce STEMSIM, a user-friendly command-line simulator of short-term evolutionary mutations for longitudinal metagenomic data"
- [methods] stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data: "stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data"
- [methods] The 'camisim' model is designed for the simulated microbial communities by CAMISIM.: "The "camisim" model is designed for the simulated microbial communities by CAMISIM."
- [methods] SnpEff (tested with 5.1, optional, annotate the vcf output of true mutations): "SnpEff (tested with 5.1, optional, annotate the vcf output of true mutations)"
- [readme] although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes."
- [readme] Three formats (.gtf, .gff, and .bed) of genomic-region file are acceptable.: "Three formats (.gtf, .gff, and .bed) of genomic-region file are acceptable."
