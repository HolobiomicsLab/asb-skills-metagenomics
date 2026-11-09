---
name: metagenomic-protein-assembly
description: Use when the goal is to maximize protein sequence recovery from complex
  metagenomic or metatranscriptomic short-read data (paired-end FASTQ or single-end
  FASTA/FASTQ) by assembling directly at the protein level.
when_to_use_negative:
- The analysis target is strain-resolved nucleotide-level reconstruction of non-coding
  regions such as 16S rRNA or complete viral genomes and protein-level assembly alone
  would miss them — use PenguiN guided_nuclassemble instead
- Reads are shorter than ~120 nt, since the default --min-length of 40 codons requires
  reads longer than 120 nt
- A nucleotide reference-guided or mapping-based approach is required because de novo
  assembly of very complex metagenomes is not the goal
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3174
tools:
- name: Plass
  role: 'Protein-level assembler: six-frame-translates reads and assembles them into
    protein sequences via `plass assemble`'
  repo: https://github.com/soedinglab/plass
- name: PenguiN
  role: Protein-guided nucleotide assembler (`penguin guided_nuclassemble` / `nuclassemble`)
    for nucleotide contigs such as viral genomes and 16S rRNAs
  repo: https://github.com/soedinglab/plass
- name: Megahit
  role: State-of-the-art nucleotide baseline assembler used for benchmarking PenguiN's
    viral genome completeness
- name: SPAdes
  role: Second nucleotide baseline assembler (SPAdes variants) used in the PenguiN
    benchmark
- name: conda
  role: Installation channel for Plass/PenguiN from bioconda
  repo: https://github.com/conda-forge/miniforge
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:c4a81cbe03bf8695150637d0a8625c07e382a350e4c0b9e41ef70b7ab2b4fc00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - Megahit
  - PenguiN
  - Plass
  - SPAdes
  - conda
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metagenomic-protein-assembly

## Summary

De novo assembly of short metagenomic/metatranscriptomic reads at the protein level using Plass, which six-frame-translates sequencing reads and assembles the translated sequences into protein sequences, recovering 2–10 times more protein sequences from complex metagenomes than nucleotide-level assemblers.

## When to use

Use when the goal is to maximize protein sequence recovery from complex metagenomic or metatranscriptomic short-read data (paired-end FASTQ or single-end FASTA/FASTQ) by assembling directly at the protein level.

## When NOT to use

- The analysis target is strain-resolved nucleotide-level reconstruction of non-coding regions such as 16S rRNA or complete viral genomes and protein-level assembly alone would miss them — use PenguiN guided_nuclassemble instead
- Reads are shorter than ~120 nt, since the default --min-length of 40 codons requires reads longer than 120 nt
- A nucleotide reference-guided or mapping-based approach is required because de novo assembly of very complex metagenomes is not the goal

## Inputs

- Paired-end metagenomic/metatranscriptomic short reads in gzipped FASTQ format (reads_1.fastq.gz, reads_2.fastq.gz)
- Single-end reads in FASTA or FASTQ format (also accepted)

## Outputs

- assembly.fas — FASTA file of assembled protein sequences
- contigs.fas — FASTA of nucleotide contigs (PenguiN nuclassemble/guided_nuclassemble mode)
- Redundancy-filtered reference protein catalogs (e.g. 90% sequence identity at 90% coverage, as for SRC and MERC)

## How to apply

Install Plass from bioconda (`conda install -c conda-forge -c bioconda plass`) on a 64-bit Linux/macOS system with at least SSE4.1 support, or compile from source for system-optimized performance. Run `plass assemble` with the paired-end FASTQ reads (or a single FASTA/FASTQ file), an output FASTA, and a tmp working directory; Plass six-frame-translates the reads and assembles them entirely at the protein level rather than building nucleotide contigs. Tune assembly stringency via --min-seq-id (overlap sequence identity threshold) and -e (E-value threshold for overlaps), keep the default --min-length of 40 codons for ORF prediction (reads must therefore be longer than 120 nt), and switch on the --filter-proteins neural network protein filter to remove spurious ORFs. For very large datasets, distribute the run over homogeneous nodes with MPI (e.g. RUNNER="mpirun -np 42") while sharing the tmp folder across all nodes, e.g. via NFS. Validate that the output FASTA contains assembled protein sequences, noting that ORFs above 100 amino acids are expected to be mostly genuine. If nucleotide contigs (e.g. viral genomes or 16S rRNA) are the target instead, use PenguiN's `guided_nuclassemble`, which links protein-level coding assemblies across non-coding regions.

## Related tools

- **Plass** (Protein-level assembler: six-frame-translates reads and assembles them into protein sequences via `plass assemble`) — https://github.com/soedinglab/plass
- **PenguiN** (Protein-guided nucleotide assembler (`penguin guided_nuclassemble` / `nuclassemble`) for nucleotide contigs such as viral genomes and 16S rRNAs) — https://github.com/soedinglab/plass
- **Megahit** (State-of-the-art nucleotide baseline assembler used for benchmarking PenguiN's viral genome completeness)
- **SPAdes** (Second nucleotide baseline assembler (SPAdes variants) used in the PenguiN benchmark)
- **conda** (Installation channel for Plass/PenguiN from bioconda) — https://github.com/conda-forge/miniforge

## Examples

```
plass assemble reads_1.fastq.gz reads_2.fastq.gz assembly.fas tmp --min-seq-id 0.0 -e 0.001 --min-length 40 --filter-proteins 1
```

## Evaluation signals

- assembly.fas exists and is a valid FASTA file of assembled protein sequences
- Assembled ORFs respect the default --min-length of 40 codons, and ORFs above 100 amino acids are expected to be mostly genuine
- Protein sequence recovery is a 2–10 fold improvement over nucleotide-level assembly of the same reads
- For PenguiN nucleotide mode, the fold-improvement in complete viral genomes over Megahit/SPAdes baselines falls within the reported 3–40x range, and 16S rRNA recovery approaches ~6x
- Running `plass assemble` with both paired-end FASTQ inputs and a tmp directory completes without errors and the tmp folder is writable (shared via NFS if MPI-distributed)

## Limitations

- --min-length defaults to 40 codons, so reads shorter than 120 nt cannot contribute ORFs
- Both Plass and PenguiN require a 64-bit Linux/macOS system with at least SSE4.1 instruction set support (AVX2 is roughly 2x faster)
- MPI-distributed assembly requires the tmp folder to be shared between all homogeneous nodes
- PenguiN protein-guided nucleotide assembly cannot link across long non-coding regions unless guided_nuclassemble mode is used

## Evidence

- [abstract] Plass assembles six-frame-translated reads at protein level: "The open-source de novo protein-level assembler, Plass ( https://plass.mmseqs.com ), assembles six-frame-translated sequencing reads into protein sequences."
- [abstract] 2-10x more protein sequences recovered: "It recovers 2–10 times more protein sequences from complex metagenomes and can assemble huge datasets."
- [methods] plass assemble CLI example: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [methods] --min-length default and --filter-proteins: "--min-length         minimum codon length for ORF prediction (default: 40)"
- [methods] MPI and shared tmp requirement: "Both tools can be distributed over several homogeneous computers. However the `tmp` folder has to be shared between all nodes"
- [methods] PenguiN benchmark performance: "It assembles 3-40 times more complete viral genomes and six times as many 16S rRNA sequences than state of the art assemblers like Megahit"
- [methods] guided_nuclassemble module semantics: "penguin guided_nuclassemble  Assembles nucleotides using protein and nucleotide information (i:Nucleotides -> o:Nucleotides)"
- [readme] Plass/PenguiN installation from bioconda: "conda install -c conda-forge -c bioconda plass"
- [readme] Catalog redundancy reduction thresholds: "Both catalogues were redundancy reduced to 90% sequence identity at 90% coverage."
