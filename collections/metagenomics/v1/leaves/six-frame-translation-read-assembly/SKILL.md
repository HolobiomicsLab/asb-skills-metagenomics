---
name: six-frame-translation-read-assembly
description: Use when short-read metagenomic or metatranscriptomic FASTQ/FASTA data
  must be assembled de novo into protein sequences rather than nucleotide contigs.
when_to_use_negative:
- Nucleotide contigs are the required output (e.g. strain-resolved assembly of viral
  genomes or microbial 16S rRNAs) — use PenguiN (`penguin nuclassemble` or `penguin
  guided_nuclassemble`), since Plass only outputs protein sequences.
- Input is already-assembled contigs or long reads rather than short-read sequencing
  data — Plass is a short-read assembler.
- The dataset is a low-complexity, non-metagenomic sample where standard nucleotide
  assemblers suffice; the tool is meant to work best for complex metagenomic or metatranscriptomic
  datasets.
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: Plass
  role: Protein-Level ASSembler that six-frame-translates the reads and assembles
    them into protein sequences in assembly.fas
  repo: https://github.com/soedinglab/plass
- name: conda
  role: 'Installation channel for Plass: conda install -c conda-forge -c bioconda
    plass'
  repo: https://github.com/conda-forge/miniforge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:e072ca9cd3829b3d46139e083ff96cc7a0a05ceee286c9dad1a389e86873f5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - Plass
  - conda
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# six-frame-translation-read-assembly

## Summary

De novo protein-level assembly: Plass six-frame-translates short sequencing reads and assembles the translated sequences directly into protein FASTA, recovering 2–10 times more protein sequences from complex metagenomic or metatranscriptomic samples than nucleotide-level assembly.

## When to use

Use when short-read metagenomic or metatranscriptomic FASTQ/FASTA data must be assembled de novo into protein sequences rather than nucleotide contigs.

## When NOT to use

- Nucleotide contigs are the required output (e.g. strain-resolved assembly of viral genomes or microbial 16S rRNAs) — use PenguiN (`penguin nuclassemble` or `penguin guided_nuclassemble`), since Plass only outputs protein sequences.
- Input is already-assembled contigs or long reads rather than short-read sequencing data — Plass is a short-read assembler.
- The dataset is a low-complexity, non-metagenomic sample where standard nucleotide assemblers suffice; the tool is meant to work best for complex metagenomic or metatranscriptomic datasets.

## Inputs

- Paired-end short-read FASTQ files, optionally gzipped (e.g. reads_1.fastq.gz, reads_2.fastq.gz)
- Single-end short reads in FASTA or FASTQ format

## Outputs

- assembly.fas — a FASTA file of assembled protein sequences
- tmp working directory holding intermediate assembly files (shared across nodes for MPI runs)

## How to apply

Run `plass assemble` with the paired-end FASTQ files (e.g. reads_1.fastq.gz and reads_2.fastq.gz), the output FASTA (assembly.fas), and a tmp working directory; single-end FASTA or FASTQ reads are handled by passing one read file. Plass works entirely at the protein level: it six-frame-translates every input read and assembles the translated sequences by overlap detection, with overlap stringency controlled by --min-seq-id (overlap sequence identity threshold) and -e (E-value threshold for overlaps). ORF prediction on the assembled translations applies --min-length (default: 40 codons), and the --filter-proteins neural network protein filter should be switched on to remove spurious proteins; ORFs above 100 amino acids are expected to be mostly genuine. For very large datasets (e.g. the paper's 640-sample soil or 775-sample TARA marine eukaryotic catalogs), assembly can be MPI-distributed over several homogeneous computers provided the tmp folder is shared between all nodes.

## Related tools

- **Plass** (Protein-Level ASSembler that six-frame-translates the reads and assembles them into protein sequences in assembly.fas) — https://github.com/soedinglab/plass
- **conda** (Installation channel for Plass: conda install -c conda-forge -c bioconda plass) — https://github.com/conda-forge/miniforge

## Examples

```
plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp
```

## Evaluation signals

- assembly.fas exists and parses as a valid FASTA file of assembled protein sequences (amino-acid alphabet, no nucleotide residues)
- Both paired-end input files (reads_1.fastq.gz and reads_2.fastq.gz) are consumed and the tmp directory is populated during the run
- No assembled protein is shorter than the --min-length default of 40 codons, and ORFs above 100 amino acids should predominate as mostly genuine
- Protein sequence recovery from the same reads should exceed nucleotide-first assembly by the reported 2–10× margin on complex metagenomes
- Running with --filter-proteins switched on removes neural-network-flagged spurious proteins relative to an unfiltered run

## Limitations

- Plass never emits nucleotide contigs — assembly happens only at the protein level on six-frame-translated reads; nucleotide assembly is PenguiN's role.
- ORFs between the 40-codon default minimum and ~100 aa may include spurious predictions (ORFs above 100 amino acids are expected to be mostly genuine), so keep the --filter-proteins neural network protein filter enabled.
- MPI-distributed assembly requires the tmp folder to be shared between all nodes and only works over several homogeneous computers.

## Evidence

- [abstract] Plass mechanism: protein-level assembly of translated reads: "assembles six-frame-translated sequencing reads into protein sequences"
- [abstract] Recovery gain motivating the protein-level approach: "It recovers 2–10 times more protein sequences from complex metagenomes and can assemble huge datasets."
- [methods] Paired-end invocation pattern: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [methods] ORF minimum length filter parameter: "--min-length         minimum codon length for ORF prediction (default: 40)"
- [readme] Scope: Plass targets short reads from complex metagenomic/metatranscriptomic data: "software to assemble protein sequences or DNA/RNA contigs from short read sequencing data meant to work best for complex metagenomic or metatranscriptomic datasets"
