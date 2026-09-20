---
name: camisim-output-parsing
description: Use when the input is a CAMISIM output directory of simulated longitudinal
  metagenomic reads and a downstream tool needs the community reads plus a genome_id-to-FASTA
  mapping table.
when_to_use_negative:
- Input reads come from a single reference genome simulated by ART or another read
  simulator — use STEMSIM's 'reads' mode with --input_reads and --reference_genome
  instead of camisim-mode parsing
- The metagenomic data are real (non-simulated) rather than CAMISIM-simulated community
  output; STEMSIM warns it only works well on species with high depth and well-characterized
  genomes
- No CAMISIM output directory or genome_to_id.tsv exists and rebuilding it is not
  feasible — the camisim mode requires both artifacts
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: STEMSim
  role: Consumes the CAMISIM output directory and genome_to_id.tsv in 'camisim' mode
    to generate longitudinal within-strain mutations on the simulated raw reads
  repo: https://github.com/BoyanZhou/STEMSim
- name: CAMISIM
  role: Produces the input directory of simulated longitudinal shotgun metagenomic
    reads and the genome_to_id.tsv mapping
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: Bowtie2
  role: Used internally by STEMSim to align the longitudinal CAMISIM-simulated reads
    to each reference genome
- name: samtools
  role: Processes the Bowtie2 alignments during the camisim-mode mutation workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:e420d9d2c0a447d96b691cfb6103b9469fb7e88a38e1c8fc3324186858a30949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - Bowtie2
  - CAMISIM
  - Python
  - STEMSim
  - numpy
  - samtools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# camisim-output-parsing

## Summary

Parse and prepare a CAMISIM output directory (simulated longitudinal raw reads of a microbial community) plus its genome_to_id.tsv mapping so that downstream tools such as STEMSIM in 'camisim' mode can consume it as input.

## When to use

Use when the input is a CAMISIM output directory of simulated longitudinal metagenomic reads and a downstream tool needs the community reads plus a genome_id-to-FASTA mapping table.

## When NOT to use

- Input reads come from a single reference genome simulated by ART or another read simulator — use STEMSIM's 'reads' mode with --input_reads and --reference_genome instead of camisim-mode parsing
- The metagenomic data are real (non-simulated) rather than CAMISIM-simulated community output; STEMSIM warns it only works well on species with high depth and well-characterized genomes
- No CAMISIM output directory or genome_to_id.tsv exists and rebuilding it is not feasible — the camisim mode requires both artifacts

## Inputs

- CAMISIM output directory containing simulated longitudinal raw sequencing reads (paired FASTQ) of a microbial community
- CAMISIM genome_to_id.tsv table mapping each genome_id to the absolute path of its reference FASTA
- STEMSim config.txt parameter file (template provided in the repository)
- Optional: nonmutated_region_file_list.txt listing genome IDs and their .gtf/.gff/.bed non-mutated region files

## Outputs

- Validated input directory path consumable by stemsim.py -m camisim (--input_from_camisim)
- genome_to_id.tsv with one Tab-separated genome_id and absolute FASTA path per line
- Downstream: modified longitudinal read sets with injected mutations and a VCF of true mutations (produced by STEMSim)

## How to apply

Locate the CAMISIM output directory that holds the simulated longitudinal raw sequencing reads of the microbial community. Extract or construct the genome_to_id.tsv table required by STEMSIM's camisim mode: each line contains a genome_id (e.g. Genome1.0) and the absolute path to its reference FASTA, separated by a Tab. Point the downstream tool at the CAMISIM output directory via a flag such as --input_from_camisim and at genome_to_id.tsv via --genome_to_id; STEMSim then aligns the longitudinal reads to each reference genome with Bowtie2 and processes the alignments with samtools/pysam to inject SNVs and small InDels across time points. Optionally supply a --nonmutated_list file (genome ID plus a .gtf/.gff/.bed genomic-region file per line) so specific regions are excluded from mutation. The rationale is that CAMISIM is a well-established pipeline for microbial community simulation, and its output directory structure plus genome_to_id.tsv is the canonical interface into camisim-mode mutation simulation.

## Related tools

- **STEMSim** (Consumes the CAMISIM output directory and genome_to_id.tsv in 'camisim' mode to generate longitudinal within-strain mutations on the simulated raw reads) — https://github.com/BoyanZhou/STEMSim
- **CAMISIM** (Produces the input directory of simulated longitudinal shotgun metagenomic reads and the genome_to_id.tsv mapping) — https://github.com/CAMI-challenge/CAMISIM
- **Bowtie2** (Used internally by STEMSim to align the longitudinal CAMISIM-simulated reads to each reference genome)
- **samtools** (Processes the Bowtie2 alignments during the camisim-mode mutation workflow)

## Examples

```
python stemsim.py -m camisim -f config.txt -s test_subject1 --input_from_camisim <CAMISIM_output_dir>/ --genome_to_id <path>/genome_to_id.tsv -o <STEMSim_output_dir>/ -l stemsim_log
```

## Evaluation signals

- genome_to_id.tsv parses cleanly with Tab separators, and every listed FASTA path exists and is absolute (e.g. 'Genome1.0\tPATH/Bifidobacterium_breve_12L.fas')
- The CAMISIM output directory contains read files for all longitudinal samples of the simulated community
- The downstream STEMSim camisim-mode run completes without errors and its output directory contains modified read files plus a VCF of injected mutations
- The mutation record lists SNV/InDel positions consistent with the configured mutation parameters for each longitudinal sample, and no mutations fall in regions listed in --nonmutated_list

## Limitations

- CAMISIM is only necessary for 'camisim' mode; using its output is recommended but the parsed artifacts are specific to CAMISIM's directory layout and genome_to_id.tsv format
- Although STEMSim can also work on real metagenomic data, it only works well on species with high depth and well-characterized genomes
- Only .gtf, .gff, and .bed formats are accepted for the optional non-mutated genomic-region files

## Evidence

- [methods] stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data: "stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data"
- [readme] each line are a genome_id and a absolute path of fasta separated by Tab: "each line are a genome_id and a absolute path of fasta separated by Tab (see example below)"
- [methods] Because CAMISIM is a well-established pipeline for the simulation of microbial community.: "Because CAMISIM is a well-established pipeline for the simulation of microbial community."
- [methods] The "camisim" model is designed for the simulated microbial communities by CAMISIM.: "The "camisim" model is designed for the simulated microbial communities by CAMISIM."
