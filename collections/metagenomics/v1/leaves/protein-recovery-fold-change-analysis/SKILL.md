---
name: protein-recovery-fold-change-analysis
description: Use when comparing protein sequence recovery of a protein-level assembler
  against nucleotide-level baselines on metagenomic reads, or when validating a manyfold
  recovery-improvement claim.
when_to_use_negative:
- The goal is nucleotide-level metrics such as complete viral genomes or 16S rRNA
  recovery — PenguiN, not Plass, is the tool that assembles 3-40 times more complete
  viral genomes and six times as many 16S rRNA sequences than Megahit and SPAdes variants.
- Input is a single-end read set where paired-end overlap information is unavailable
  and the paired-end assembly comparison design cannot be reproduced.
- The reads are not from a complex metagenomic or metatranscriptomic sample; the 2-10x
  recovery claim was established on complex metagenomes and may not transfer.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Plass
  role: Protein-level assembler run in paired-end mode with the neural network protein
    filter (--filter-proteins) to produce the protein sequences being benchmarked
  repo: https://github.com/soedinglab/plass
- name: Megahit
  role: Nucleotide-level de novo assembler providing one baseline set of contigs for
    ORF prediction and recovery comparison
- name: SPAdes
  role: SPAdes variant nucleotide assembler providing the second baseline set of contigs
    for the recovery comparison
- name: conda
  role: Installation channel for Plass and the baseline assemblers (conda install
    -c conda-forge -c bioconda plass)
  repo: https://github.com/conda-forge/miniforge
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:986c0b48379b5159cb25b4127c1813f3703dcabef981fb988bcb6d46c2b61509
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# protein-recovery-fold-change-analysis

## Summary

Benchmarks protein-level assembly (Plass) against nucleotide-level assemblers (Megahit, SPAdes) to quantify how many times more non-redundant protein sequences Plass recovers from complex metagenomes, the paper's central 2-10x recovery claim.

## When to use

Use when comparing protein sequence recovery of a protein-level assembler against nucleotide-level baselines on metagenomic reads, or when validating a manyfold recovery-improvement claim.

## When NOT to use

- The goal is nucleotide-level metrics such as complete viral genomes or 16S rRNA recovery — PenguiN, not Plass, is the tool that assembles 3-40 times more complete viral genomes and six times as many 16S rRNA sequences than Megahit and SPAdes variants.
- Input is a single-end read set where paired-end overlap information is unavailable and the paired-end assembly comparison design cannot be reproduced.
- The reads are not from a complex metagenomic or metatranscriptomic sample; the 2-10x recovery claim was established on complex metagenomes and may not transfer.

## Inputs

- Paired-end metagenomic sequencing reads (FASTQ, e.g. reads_1.fastq.gz, reads_2.fastq.gz)
- Nucleotide contigs from Megahit assembly
- Nucleotide contigs from SPAdes-variant assembly
- Protein sequences from plass assemble output (FASTA)

## Outputs

- Non-redundant protein sequence counts per assembler (Plass, Megahit, SPAdes)
- Plass fold-improvement over each nucleotide-level baseline
- recovery_comparison.csv
- Redundancy-reduced (90% identity, 90% coverage) protein catalogues per assembler

## How to apply

Assemble the same paired-end FASTQ reads twice: once with `plass assemble` (keeping the default --min-length of 40 codons, default --min-seq-id overlap identity and -e overlap E-value thresholds, and the neural network protein filter via --filter-proteins) and once with Megahit and a SPAdes variant to obtain nucleotide contigs. Predict and translate ORFs from the nucleotide contigs, retaining proteins above 100 amino acids since assemblies above 100 amino acids are mostly genuine. Redundancy-reduce each protein set to 90% sequence identity at 90% coverage, matching the convention used for the Plass reference catalogues (SRC, MERC). Count the non-redundant protein sequences from each assembler and compute Plass's fold-improvement over each baseline; a 2-10x increase on complex metagenomes is consistent with the reported finding.

## Related tools

- **Plass** (Protein-level assembler run in paired-end mode with the neural network protein filter (--filter-proteins) to produce the protein sequences being benchmarked) — https://github.com/soedinglab/plass
- **Megahit** (Nucleotide-level de novo assembler providing one baseline set of contigs for ORF prediction and recovery comparison)
- **SPAdes** (SPAdes variant nucleotide assembler providing the second baseline set of contigs for the recovery comparison)
- **conda** (Installation channel for Plass and the baseline assemblers (conda install -c conda-forge -c bioconda plass)) — https://github.com/conda-forge/miniforge

## Examples

```
conda install -c conda-forge -c bioconda plass; plass assemble reads_1.fastq.gz reads_2.fastq.gz plass_assembly.fas tmp --filter-proteins
```

## Evaluation signals

- Plass recovers 2-10 times more protein sequences than Megahit and the SPAdes variant on the complex metagenome, consistent with the reported fold-improvement.
- All three protein sets are redundancy-reduced to 90% sequence identity at 90% coverage before counting, matching the SRC/MERC catalogue convention.
- Proteins predicted from nucleotide contigs above 100 amino acids are retained (assemblies above 100 amino acids are mostly genuine); counts should be computed only on this filtered set.
- Plass assembly used default --min-length of 40 codons, default --min-seq-id and -e overlap thresholds, with --filter-proteins enabled — deviations from these parameters invalidate comparability.
- recovery_comparison.csv exists and contains per-assembler non-redundant protein counts plus computed fold-change columns.

## Limitations

- The 2-10x recovery improvement was measured on complex metagenomes; results on low-complexity or isolate datasets may differ.
- Both Plass and PenguiN are GPL-licensed C++ software for Linux and macOS designed to run on multiple cores; Windows or single-core environments are unsupported.
- Very short ORFs (< 100 amino acids) from nucleotide baselines are excluded to avoid counting spurious translations, which means the baseline comparison is restricted to proteins above 100 amino acids.
- Distributed (MPI) assembly requires the tmp folder to be shared between all nodes.

## Evidence

- [abstract] Plass recovers 2-10x more proteins than nucleotide baselines: "It recovers 2–10 times more protein sequences from complex metagenomes and can assemble huge datasets."
- [methods] Paired-end plass assemble command: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [methods] Neural network protein filter option: "--filter-proteins    Switches the neural network protein filter off/on"
- [methods] ORF minimum length default: "--min-length         minimum codon length for ORF prediction (default: 40)"
- [readme] Catalogue redundancy-reduction convention: "Both catalogues were redundancy reduced to 90% sequence identity at 90% coverage."
- [methods] Baseline assemblers named: "assemblers like Megahit and the SPAdes variants"
