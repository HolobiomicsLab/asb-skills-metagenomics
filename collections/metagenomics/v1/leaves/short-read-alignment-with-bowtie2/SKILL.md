---
name: short-read-alignment-with-bowtie2
description: Use when raw sequencing reads were simulated by software other than CAMISIM
  (e.g., ART) and must be placed against a reference genome (.fas) as input to STEMSIM's
  reads mode.
when_to_use_negative:
- Input is already the output directory of CAMISIM — use STEMSIM's 'camisim' mode
  instead, which reads CAMISIM's own genome_to_id.tsv mapping and does not require
  re-alignment
- Reads originate from more than one reference genome/species — the reads mode is
  designed for simulated raw reads from one reference genome
- Real metagenomic data from species with low depth or poorly characterized genomes
  — the README warns STEMSIM only works well on species with high depth and well-characterized
  genomes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Bowtie2
  role: Aligns input raw reads to the supplied reference genome to establish read-to-genome
    placement within STEMSIM's reads mode
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Processes Bowtie2 alignments on the reference genome, called through pysam
    inside STEMSIM
  repo: http://www.htslib.org/doc/
- name: pysam
  role: Python interface used to parse samtools alignments and establish read-to-genome
    placement
- name: STEMSim
  role: Consumes the alignments to generate longitudinal within-strain mutations and
    output mutated reads plus a VCF of true mutations
  repo: https://github.com/BoyanZhou/STEMSim
- name: ART
  role: Optional source of externally simulated paired-end reads for the reads-mode
    alignment input
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:36c3c5995249e34bb64eac430f7116e91104c9db3c37ae92def6466e4ebdf6dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie2
  - STEMSim
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# short-read-alignment-with-bowtie2

## Summary

Align externally produced raw metagenomic reads (fastq or unaligned BAM, e.g., from ART) to a supplied reference genome with Bowtie2, then process alignments with samtools via pysam so STEMSIM can establish read-to-genome placement before applying its within-strain mutation model.

## When to use

Use when raw sequencing reads were simulated by software other than CAMISIM (e.g., ART) and must be placed against a reference genome (.fas) as input to STEMSIM's reads mode.

## When NOT to use

- Input is already the output directory of CAMISIM — use STEMSIM's 'camisim' mode instead, which reads CAMISIM's own genome_to_id.tsv mapping and does not require re-alignment
- Reads originate from more than one reference genome/species — the reads mode is designed for simulated raw reads from one reference genome
- Real metagenomic data from species with low depth or poorly characterized genomes — the README warns STEMSIM only works well on species with high depth and well-characterized genomes

## Inputs

- Raw sequencing reads (fastq or unaligned bam) from an external simulator such as ART, formatted as PATH/sample0_R1.fq,PATH/sample0_R2.fq:PATH/sample1.fq:PATH/sample2.fq (paired mates by ',', samples by ':')
- Reference genome in .fas format (the genome from which reads were simulated)
- Genome ID string for the reference genome (--genome_id)
- STEMSim template config file (config.txt)

## Outputs

- Read-to-genome placement of input reads (Bowtie2/samtools alignments parsed via pysam) used internally by STEMSIM
- Mutated reads matching the input read structure, written to the STEMSIM output directory
- VCF file of true mutations, optionally annotated with SnpEff 5.1
- Populated STEMSIM log file

## How to apply

In STEMSIM's reads mode, run Bowtie2 (tested with 0.16.0.1) to align each longitudinal input read file — with paired mates joined by ',' and samples ordered by ':' per the --input_reads format — against the reference genome supplied via --reference_genome (a .fas file). Process the resulting alignments with samtools (tested with 1.9) through pysam (0.16.0.1) to establish read-to-genome placement, which determines where the simulated short-term evolutionary mutations are applied. The rationale is that non-CAMISIM reads lack CAMISIM's internal genome-to-read mapping, so alignment reconstructs that placement before the mutation model writes modified reads plus the VCF of true mutations. Invocation requires -m reads, the template config.txt (-f), a subject ID (-s), an output directory (-o), --genome_id, and a log file (-l).

## Related tools

- **Bowtie2** (Aligns input raw reads to the supplied reference genome to establish read-to-genome placement within STEMSIM's reads mode) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Processes Bowtie2 alignments on the reference genome, called through pysam inside STEMSIM) — http://www.htslib.org/doc/
- **pysam** (Python interface used to parse samtools alignments and establish read-to-genome placement)
- **STEMSim** (Consumes the alignments to generate longitudinal within-strain mutations and output mutated reads plus a VCF of true mutations) — https://github.com/BoyanZhou/STEMSim
- **ART** (Optional source of externally simulated paired-end reads for the reads-mode alignment input) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m reads -f PATH/config.txt -s test_subject1 --input_reads PATH/sample0_R1.fq,PATH/sample0_R2.fq:PATH/sample1.fq --reference_genome PATH/species0_strain0.fas --genome_id species0_strain0 -o PATH_to_output_directory_of_STEMSim/ -l Path/stemsim_test1_log
```

## Evaluation signals

- Bowtie2 completes alignment for every longitudinal sample in the --input_reads string and alignments are parseable by pysam 0.16.0.1
- The output directory contains a mutated read set matching the structure of the input reads (same number of files/samples)
- A VCF of true mutations is present alongside the mutated reads
- The run completes with the specified log file (-l) populated
- The reads-mode command was invoked with --reference_genome (.fas) and --genome_id matching the genome the reads were simulated from

## Limitations

- Bowtie2 placement is only needed because non-CAMISIM reads lack CAMISIM's internal mapping; the README recommends CAMISIM output as input because CAMISIM is a well-established pipeline
- Although STEMSIM can work on real metagenomic data, it only works well on species with high depth and well-characterized genomes
- Versions are pinned by testing only: Bowtie2 0.16.0.1, samtools 1.9, pysam 0.16.0.1; untested versions may behave differently

## Evidence

- [other] Within STEMSIM, align the input reads to the supplied reference genome with Bowtie2 (tested with 0.16.0.1) and process alignments with samtools: [evidence span withheld — anchor status: not_found]
- [readme] reads mode takes external reads: "stemsim takes the simulated raw reads from other software (such as ART) as input, and generate longitudinal mutations on the simulated raw sequencing data"
- [readme] reads-mode input format: "input raw read files from longitudinal samples separated by ':'; for paired reads from the same sample, they are separated by ','"
