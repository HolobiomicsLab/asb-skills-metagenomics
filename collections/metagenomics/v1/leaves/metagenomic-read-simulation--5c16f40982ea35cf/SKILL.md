---
name: metagenomic-read-simulation
description: Use when you need synthetic longitudinal metagenomic reads carrying within-strain
  SNVs and small InDels with configurable allele-frequency trajectories, to benchmark
  mutation-detection tools.
when_to_use_negative:
- Data are real metagenomic samples from low-depth or poorly characterized species
  — STEMSim only works well on species with high depth and well-characterized genomes
- Goal is community-level abundance simulation rather than within-strain short-term
  evolutionary mutations — use CAMISIM alone to generate the input community
- Non-longitudinal data with no within-strain evolutionary signal to model across
  timepoints
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: STEMSim
  role: 'Main simulator: generates longitudinal SNVs/small InDels on raw reads and
    emits modified reads plus a VCF of true mutations'
  repo: https://github.com/BoyanZhou/STEMSim
- name: Bowtie2
  role: Aligns longitudinal reads to each reference genome internally before mutation
    injection
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Processes alignments (tested with 1.9) during read-to-genome placement
  repo: http://www.htslib.org/doc/
- name: CAMISIM
  role: Optional upstream simulator whose output directory is the input for camisim
    mode
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: ART
  role: Optional external reads simulator whose FASTQ output feeds reads mode
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- name: SnpEff
  role: Optional annotation of the VCF output of true mutations (tested with 5.1)
  repo: http://pcingola.github.io/SnpEff/
- name: pysam
  role: Python package (tested with 0.16.0.1) used with numpy for alignment processing
    inside STEMSim
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:715e729a75c3ba26ad0195503c5e83d788e9381af380a1236a56e38c94b988bf
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# metagenomic-read-simulation

## Summary

Simulate within-strain short-term evolutionary mutations on longitudinal metagenomic raw reads using STEMSIM, in either camisim mode (CAMISIM output directory as input) or reads mode (externally simulated FASTQs with a supplied reference genome), producing modified reads plus a VCF of true injected mutations.

## When to use

Use when you need synthetic longitudinal metagenomic reads carrying within-strain SNVs and small InDels with configurable allele-frequency trajectories, to benchmark mutation-detection tools.

## When NOT to use

- Data are real metagenomic samples from low-depth or poorly characterized species — STEMSim only works well on species with high depth and well-characterized genomes
- Goal is community-level abundance simulation rather than within-strain short-term evolutionary mutations — use CAMISIM alone to generate the input community
- Non-longitudinal data with no within-strain evolutionary signal to model across timepoints

## Inputs

- CAMISIM output directory of simulated longitudinal raw reads (camisim mode)
- genome_to_id.tsv mapping each genome_id to its absolute FASTA path (camisim mode)
- Longitudinal raw read FASTQs (or unaligned BAM) from other simulators such as ART, samples separated by ':' and paired mates by ',' (reads mode)
- Reference genome FASTA (.fas) and genome id (reads mode)
- STEMSim config.txt parameter file (substitution model, mutation frequencies per timepoint)
- Optional non-mutated genomic-region files (.gtf, .gff, or .bed)
- Optional SnpEff directory and genomeID-to-snpEff ID matching table for VCF annotation

## Outputs

- Modified (mutated) raw metagenomic read sets written to the output directory
- VCF file of injected true mutations (optionally annotated with SnpEff 5.1)
- Run log file (e.g. stemsim_test1_log)

## How to apply

Choose the working mode based on input origin: 'camisim' mode takes a CAMISIM output directory plus its genome_to_id.tsv table (genome_id to absolute FASTA path), while 'reads' mode takes fastq (or unaligned bam) reads simulated by other software such as ART, with samples separated by ':' and paired mates of one sample separated by ',', plus --reference_genome (.fas) and --genome_id. Invoke stemsim.py with -m camisim|reads -f config.txt -s subject_id, the mode-appropriate input flags, -o output directory and -l log file; internally STEMSim aligns longitudinal reads to each reference genome with Bowtie2 and processes alignments with samtools/pysam, then samples SNVs and small InDels across timepoints (selecting read carriers via Bernoulli sampling at configured per-timepoint frequencies, e.g. 0.2/0.5/0.8/1 for positive selection or 0.8/0.5/3/0-style decreasing trajectories) while excluding any regions listed in --nonmutated_list/--nonmutated_file (.gtf/.gff/.bed). SNV spectra can be drawn from five selectable substitution models (JC69, K80, HKY85, TN93, REV) set in config.txt (template parameters documented at the STEMSim wiki). Optionally annotate the VCF of true mutations with SnpEff by supplying -p (snpEff directory) and -a (genomeID-to-snpEff ID matching table). The rationale is that STEMSIM fills the gap of tools simulating within-strain evolutionary signals, enabling evaluation of analytic tools that detect short-term evolutionary mutations in metagenomic data.

## Related tools

- **STEMSim** (Main simulator: generates longitudinal SNVs/small InDels on raw reads and emits modified reads plus a VCF of true mutations) — https://github.com/BoyanZhou/STEMSim
- **Bowtie2** (Aligns longitudinal reads to each reference genome internally before mutation injection) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Processes alignments (tested with 1.9) during read-to-genome placement) — http://www.htslib.org/doc/
- **CAMISIM** (Optional upstream simulator whose output directory is the input for camisim mode) — https://github.com/CAMI-challenge/CAMISIM
- **ART** (Optional external reads simulator whose FASTQ output feeds reads mode) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- **SnpEff** (Optional annotation of the VCF output of true mutations (tested with 5.1)) — http://pcingola.github.io/SnpEff/
- **pysam** (Python package (tested with 0.16.0.1) used with numpy for alignment processing inside STEMSim)

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m camisim -f PATH/config.txt -s test_subject1 --input_from_camisim PATH_to_output_directory_of_CAMISIM/ --genome_to_id PATH_to_genome_to_id/genome_to_id.tsv -o PATH_to_output_directory_of_STEMSim/ -l Path/stemsim_test1_log
```

## Evaluation signals

- Run completes without errors and the output directory contains modified read files plus a VCF of injected mutations, with the -l log file populated
- VCF mutation positions and SNV/InDel counts are consistent with the configured mutation parameters and the selected substitution model (JC69/K80/HKY85/TN93/REV) per genome
- Observed per-timepoint allele frequencies extracted from aligned mutated reads match the configured Bernoulli trajectories (e.g. increasing 0.2/0.5/0.8/1)
- Mutated read set structure matches the input (paired mates and sample ordering preserved)
- Throughput sanity check: ~1 million reads processed in approximately three minutes on a 2.40 GHz server CPU

## Limitations

- Although STEMSim can work on real metagenomic data, it only works well on species with high depth and well-characterized genomes
- Tested dependency versions are specific: Python 3.6.5, numpy 1.19.5, pysam 0.16.0.1, Bowtie2 (0.16.0.1 as tested), samtools 1.9
- CAMISIM is only necessary for camisim mode; reads mode requires the original reference genome to be supplied alongside non-CAMISIM reads

## Evidence

- [abstract] camisim mode input: "we introduce STEMSIM, a user-friendly command-line simulator of short-term evolutionary mutations for longitudinal metagenomic data"
- [methods] modes: "The "camisim" model is designed for the simulated microbial communities by CAMISIM."
- [readme] reads mode: "Under this mode, stemsim takes the simulated raw reads from other software (such as ART) as input, and generate longitudinal mutations on the simulated raw sequencing data."
- [readme] input_reads format: "is input raw read files from longitudinal samples separated by ':'; for paired reads from the same sample, they are separated by ','"
- [readme] limitations: "it only works well on species with high depth and well-characterized genomes"
- [readme] nonmutated regions: "Three formats (.gtf, .gff, and .bed) of genomic-region file are acceptable. For each genome listed, no mutation will be generated in the genomic regions listed"
