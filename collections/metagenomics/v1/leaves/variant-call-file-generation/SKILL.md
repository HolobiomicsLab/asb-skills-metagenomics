---
name: variant-call-file-generation
description: 'Use when simulated longitudinal metagenomic reads must carry ground-truth
  variants: STEMSIM writes mutated reads plus a VCF detailing the exact mutations
  it introduced.'
when_to_use_negative:
- You need to call variants from real, experimentally derived metagenomic reads for
  discovery rather than simulation ground truth — STEMSIM only works well on species
  with high depth and well-characterized genomes
- Input data are already CAMISIM output with an existing simulated community; the
  'camisim' mode with --input_from_camisim and --genome_to_id.tsv is the appropriate
  path instead
- The goal is de novo variant discovery without a supplied reference genome; the reads
  mode requires --reference_genome and --genome_id to place reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: STEMSIM
  role: Core simulator that applies the within-strain evolutionary mutation model
    and writes the VCF of true mutations plus modified reads
  repo: https://github.com/BoyanZhou/STEMSim
- name: Bowtie2
  role: Aligns input reads to the supplied reference genome to establish read-to-genome
    placement before mutation simulation
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Processes Bowtie2 alignments (via pysam) to support mutation placement
  repo: http://www.htslib.org/doc/
- name: pysam
  role: Python interface used by STEMSIM to process samtools alignment records
  repo: https://github.com/BoyanZhou/STEMSim
- name: ART
  role: Optional external reads simulator that can generate the fastq input consumed
    by the 'reads' mode
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- name: SnpEff
  role: Optionally annotates the VCF output of true mutations when the snpEff directory
    and ID-matching table are provided
  repo: http://pcingola.github.io/SnpEff/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:d0a83c400dd8fc496d3d8b8b4ad3876cbb1f304f964dcc283294ebd4117a0c6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - ART
  - Bowtie2
  - STEMSIM
  - SnpEff
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# variant-call-file-generation

## Summary

Generation of a VCF of true mutations produced when simulating within-strain short-term evolutionary mutations on longitudinal metagenomic reads with STEMSIM, optionally annotated with SnpEff.

## When to use

Use when simulated longitudinal metagenomic reads must carry ground-truth variants: STEMSIM writes mutated reads plus a VCF detailing the exact mutations it introduced.

## When NOT to use

- You need to call variants from real, experimentally derived metagenomic reads for discovery rather than simulation ground truth — STEMSIM only works well on species with high depth and well-characterized genomes
- Input data are already CAMISIM output with an existing simulated community; the 'camisim' mode with --input_from_camisim and --genome_to_id.tsv is the appropriate path instead
- The goal is de novo variant discovery without a supplied reference genome; the reads mode requires --reference_genome and --genome_id to place reads

## Inputs

- Longitudinal raw sequencing reads in fastq or unaligned bam format (paired mates joined by ',', samples ordered by ':'), from CAMISIM or other simulators such as ART
- Reference genome in .fas format from which the reads were simulated, supplied via --reference_genome
- Genome identifier string supplied via --genome_id
- STEMSim template config file (config.txt) with mutation-model parameters
- Optional genomic-region file (.gtf, .gff, or .bed) listing regions in which no mutation should be generated (--nonmutated_file / --nonmutated_list)
- Optional snpEff directory and genomeID-to-snpEff ID-matching table for VCF annotation

## Outputs

- Modified (mutated) read set matching the structure of the input reads, written to the output directory
- VCF file of true (ground-truth) mutations introduced by the simulation
- Optionally, a SnpEff-annotated version of the mutation VCF
- Populated STEMSIM log file

## How to apply

Run stemsim.py in 'reads' mode (or 'camisim' mode) with a template config file (-f), a subject ID (-s), input reads (--input_reads with paired mates joined by ',' and longitudinal samples ordered by ':'), the reference genome (.fas) via --reference_genome, and --genome_id. STEMSIM aligns input reads to the supplied reference genome with Bowtie2 (tested 0.16.0.1) and processes alignments with samtools (1.9) through pysam (0.16.0.1) to establish read-to-genome placement, then applies its within-strain short-term evolutionary mutation model to generate longitudinal mutations. The tool writes both the modified reads and a VCF of true mutations to the output directory (-o) with progress recorded in the log file (-l). Optionally pass the SnpEff directory (-p) and an ID-matching table (-a) so the VCF is annotated with SnpEff 5.1.

## Related tools

- **STEMSIM** (Core simulator that applies the within-strain evolutionary mutation model and writes the VCF of true mutations plus modified reads) — https://github.com/BoyanZhou/STEMSim
- **Bowtie2** (Aligns input reads to the supplied reference genome to establish read-to-genome placement before mutation simulation) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Processes Bowtie2 alignments (via pysam) to support mutation placement) — http://www.htslib.org/doc/
- **pysam** (Python interface used by STEMSIM to process samtools alignment records) — https://github.com/BoyanZhou/STEMSim
- **ART** (Optional external reads simulator that can generate the fastq input consumed by the 'reads' mode) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm
- **SnpEff** (Optionally annotates the VCF output of true mutations when the snpEff directory and ID-matching table are provided) — http://pcingola.github.io/SnpEff/

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m reads -f PATH/config.txt -s test_subject1 --input_reads PATH/sample0_R1.fq,PATH/sample0_R2.fq:PATH/sample1.fq -o PATH_out/ --reference_genome PATH/species0_strain0.fas --genome_id species0_strain0 -l Path/stemsim_test1_log
```

## Evaluation signals

- The output directory contains a VCF of true mutations alongside a mutated read set matching the structure of the input reads
- The run completes with the specified log file populated
- Each entry in the VCF corresponds to a mutation actually introduced in the modified reads (true ground-truth variants, not called variants)
- If --nonmutated_file is given, no mutations appear in the listed genomic regions (.gtf/.gff/.bed)
- If SnpEff annotation was requested (-p and -a), VCF entries carry SnpEff annotations matched via the genomeID-to-snpEff ID table

## Limitations

- STEMSim works well on species with high depth and well-characterized genomes; performance on poorly characterized genomes is not guaranteed
- Although it can run on real metagenomic data, STEMSim is designed for longitudinal or concurrent simulated raw sequencing reads of microbial communities
- SnpEff annotation is optional and requires an ID-matching table mapping genome IDs to snpEff database IDs

## Evidence

- [abstract] STEMSIM outputs modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: [evidence span withheld — anchor status: not_found]
- [methods] stemsim takes the simulated raw reads from other software (such as ART) as input: "stemsim takes the simulated raw reads from other software (such as ART) as input, and generate longitudinal mutations on the simulated raw sequencing data"
- [methods] SnpEff (tested with 5.1, optional, annotate the vcf output of true mutations): "SnpEff (tested with 5.1, optional, annotate the vcf output of true mutations)"
- [readme] -o PATH_to_output_directory_of_STEMSim/ is the output directory of STEMSim;: "-o PATH_to_output_directory_of_STEMSim/ is the output directory of STEMSim;"
- [readme] it only works well on species with high depth and well-characterized genomes.: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes."
