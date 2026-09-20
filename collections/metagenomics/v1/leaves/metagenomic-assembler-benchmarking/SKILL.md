---
name: metagenomic-assembler-benchmarking
description: Use when comparing how many protein sequences different de novo assemblers
  recover from the same metagenomic reads, or when reproducing the Plass 2–10x recovery
  claim on complex metagenomes.
when_to_use_negative:
- The goal is strain-resolved nucleotide assembly of viral genomes or 16S rRNAs —
  use PenguiN instead, which assembles 3–40x more complete viral genomes and 6x more
  16S rRNA sequences than Megahit/SPAdes
- Input is already assembled contigs or a protein catalogue — benchmarking requires
  raw short reads run through each assembler
- The sample is a simple isolate genome rather than a complex metagenomic/metatranscriptomic
  dataset, where Plass's recovery advantage was demonstrated
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3174
tools:
- name: Plass
  role: Protein-level assembler; six-frame-translates reads and assembles them into
    protein sequences, the treatment arm of the benchmark
  repo: https://github.com/soedinglab/plass
- name: Megahit
  role: Nucleotide-level baseline assembler producing contigs whose ORFs quantify
    baseline protein recovery
- name: SPAdes
  role: Nucleotide-level baseline assembler (variant) producing contigs for the recovery
    comparison
- name: conda
  role: Installation channel for Plass via bioconda
  repo: https://github.com/conda-forge/miniforge
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:f6a2f8c3913a36e77a32924760696e53c37b0063c427f024147979ba6c4b9ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - Megahit
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
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metagenomic-assembler-benchmarking

## Summary

Benchmarks protein-level assembly (Plass) against nucleotide-level assemblers (Megahit, SPAdes variants) by comparing non-redundant protein sequence recovery from the same metagenomic reads, to quantify the reported 2–10x recovery improvement.

## When to use

Use when comparing how many protein sequences different de novo assemblers recover from the same metagenomic reads, or when reproducing the Plass 2–10x recovery claim on complex metagenomes.

## When NOT to use

- The goal is strain-resolved nucleotide assembly of viral genomes or 16S rRNAs — use PenguiN instead, which assembles 3–40x more complete viral genomes and 6x more 16S rRNA sequences than Megahit/SPAdes
- Input is already assembled contigs or a protein catalogue — benchmarking requires raw short reads run through each assembler
- The sample is a simple isolate genome rather than a complex metagenomic/metatranscriptomic dataset, where Plass's recovery advantage was demonstrated

## Inputs

- Paired-end metagenomic short reads (FASTQ, e.g. reads_1.fastq.gz / reads_2.fastq.gz)
- Optional single-end reads in FASTA or FASTQ for plass assemble

## Outputs

- Plass protein assembly FASTA (plass_assembly.fas)
- Nucleotide contigs from Megahit and SPAdes baselines
- Translated ORF protein sets (>100 aa) from baseline contigs
- recovery_comparison.csv with non-redundant protein counts and fold-improvement per assembler

## How to apply

Install Plass via bioconda (conda install -c conda-forge -c bioconda plass) or the AVX2 static build from mmseqs.com/plass, then run plass assemble in paired-end mode keeping defaults: --min-length 40 codons for ORF prediction, default --min-seq-id overlap identity and -e overlap E-value thresholds, with the neural network protein filter (--filter-proteins) enabled. Assemble the identical reads with Megahit and a SPAdes variant to obtain nucleotide contig baselines, then predict and translate ORFs from those contigs, retaining proteins above 100 amino acids since assemblies above 100 amino acids are mostly genuine. Redundancy-reduce every protein set to 90% sequence identity at 90% coverage, matching the convention used for the SRC and MERC reference catalogues. Finally count non-redundant proteins per assembler, compute Plass's fold-improvement over each baseline, write recovery_comparison.csv, and confirm the 2–10x improvement over the nucleotide-level baselines.

## Related tools

- **Plass** (Protein-level assembler; six-frame-translates reads and assembles them into protein sequences, the treatment arm of the benchmark) — https://github.com/soedinglab/plass
- **Megahit** (Nucleotide-level baseline assembler producing contigs whose ORFs quantify baseline protein recovery)
- **SPAdes** (Nucleotide-level baseline assembler (variant) producing contigs for the recovery comparison)
- **conda** (Installation channel for Plass via bioconda) — https://github.com/conda-forge/miniforge

## Examples

```
conda install -c conda-forge -c bioconda plass; plass assemble reads_1.fastq.gz reads_2.fastq.gz plass_assembly.fas tmp
```

## Evaluation signals

- Plass recovers 2–10 times more non-redundant protein sequences than Megahit and SPAdes baselines on the complex metagenome
- All compared protein sets are redundancy-reduced to 90% sequence identity at 90% coverage before counting, matching SRC/MERC catalogue convention
- Baseline ORFs shorter than 100 amino acids are excluded, since assemblies above 100 amino acids are mostly genuine
- Plass run uses --filter-proteins (neural network protein filter) and default --min-length 40, --min-seq-id, and -e overlap E-value thresholds
- recovery_comparison.csv is produced with fold-improvement computed for Plass over each baseline

## Limitations

- The 2–10x recovery improvement was demonstrated on complex metagenomes; it may not transfer to simple or low-complexity samples
- PenguiN, not Plass, is the tool for strain-resolved nucleotide assembly of viral genomes and 16S rRNAs
- Plass and PenguiN require 64-bit Linux/macOS systems; conda or static builds are the supported install routes, and MPI-distributed assembly requires a shared tmp folder between nodes

## Evidence

- [abstract] Plass recovers 2–10 times more protein sequences from complex metagenomes: "It recovers 2–10 times more protein sequences from complex metagenomes and can assemble huge datasets."
- [methods] Plass paired-end assembly invocation: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [methods] ORF minimum length default: "--min-length         minimum codon length for ORF prediction (default: 40)"
- [methods] Neural network protein filter: "--filter-proteins    Switches the neural network protein filter off/on"
- [readme] Catalogue redundancy-reduction convention: "Both catalogues were redundancy reduced to 90% sequence identity at 90% coverage."
- [abstract] Plass assembles six-frame-translated reads: "The open-source de novo protein-level assembler, Plass ( https://plass.mmseqs.com ), assembles six-frame-translated sequencing reads into protein sequences."
