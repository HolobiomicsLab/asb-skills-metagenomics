---
name: mapped-unmapped-read-fraction-comparison
description: Use when a significant proportion of long (HiFi) metagenomic reads remain
  unmapped to an assembly and you need to determine whether low abundance (e.g. median
  k-mer abundance of 1), read length, or read quality explains the unassembled fraction.
when_to_use_negative:
- The dataset is short-read or hybrid rather than HiFi long reads — Mapler's evaluation
  and fractions workflow targets HiFi metagenome assemblies
- You only need bin-level quality (completeness/contamination via CheckM2/GTDB-Tk)
  rather than explaining reads left out of the assembly
- No assembly is available and the goal is de novo assembly benchmarking rather than
  read-fraction characterization
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Mapler
  role: Snakemake pipeline that aligns reads on contigs, computes mapped read/base
    ratios, and splits reads into mapped/unmapped FASTQ fractions
  repo: https://github.com/Nimauric/Mapler
- name: minimap2
  role: Aligns the HiFi reads on contigs to define mapped vs unmapped fractions
- name: FastQC
  role: Summarizes average read length and per-base quality of each read fraction
  repo: https://github.com/s-andrews/FastQC
- name: KAT
  role: Computes k-mer abundance statistics (kat-plot.pdf, <fraction_name>-stats.tsv)
    to show whether unmapped reads are dominated by abundance-1 k-mers
  repo: https://github.com/EarlhamInst/KAT
- name: Kraken2
  role: Taxonomic assignment of each read fraction for Krona-based comparison of assignability
  repo: https://github.com/DerrickWood/kraken2
- name: metaMDBG
  role: Assembler whose assembly output (assembly.fasta) serves as the contig reference
    for the read-mapping evaluation
  repo: https://github.com/GaetanBenoitDev/metaMDBG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:91775c251f46e3c8d23984b68f1b1c011a121e4ca8fa6d58a19da1f451dde02b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Bash
  - Conda
  - FastQC
  - KAT
  - Kraken2
  - Mapler
  - Python
  - metaMDBG
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# mapped-unmapped-read-fraction-comparison

## Summary

Characterizes the mapped vs unmapped read fractions from aligning HiFi reads on a metagenome assembly (via the Mapler pipeline) to explain why reads failed to assemble — comparing read length, per-base quality, k-mer abundance (KAT), and taxonomic assignment (Kraken 2/Krona) between fractions.

## When to use

Use when a significant proportion of long (HiFi) metagenomic reads remain unmapped to an assembly and you need to determine whether low abundance (e.g. median k-mer abundance of 1), read length, or read quality explains the unassembled fraction.

## When NOT to use

- The dataset is short-read or hybrid rather than HiFi long reads — Mapler's evaluation and fractions workflow targets HiFi metagenome assemblies
- You only need bin-level quality (completeness/contamination via CheckM2/GTDB-Tk) rather than explaining reads left out of the assembly
- No assembly is available and the goal is de novo assembly benchmarking rather than read-fraction characterization

## Inputs

- HiFi long-read FASTQ read file per sample (e.g. Deadwood sample reads)
- Assembly FASTA from any assembler (metaMDBG assembly.fasta or user-submitted assembly placed at outputs/<sample_name>/<assembler>/assembly.fasta)
- Mapler configfile (copy of config/config_template.yaml) with samples, assemblers, and fastqc/kat/kraken2 set to true
- Optional: Kraken 2 database (kraken2db) and GTDB-Tk reference data paths in the configfile

## Outputs

- Mapped and unmapped read fractions as <fraction_name>_reads.fastq under outputs/<sample_name>/<assembler>/
- read_contig_mapping.txt and report.txt with aligned read and aligned base ratios
- FastQC reports (fastqc_report.html, <fraction_name>_fastqc.zip) per fraction
- KAT k-mer abundance stats (<fraction_name>-stats.tsv) and kat-plot.pdf
- Kraken 2 assignments (kraken2.tsv) and interactive Krona plots (krona.html) per fraction

## How to apply

Configure a Mapler Snakemake configfile with the sample reads and the assembly (e.g. the metaMDBG assembly.fasta from doi:10.57745/2SA8AB), enabling read-mapping/evaluation and setting fastqc, kat, and kraken2 to true. Mapler aligns reads on contigs with minimap2, computes mapped read and base ratios, and writes the mapped and unmapped fractions as <fraction_name>_reads.fastq under outputs/<sample_name>/<assembler>/. Then run FastQC on each fraction to record average read length and per-base quality (e.g. mapped vs unmapped averages of 18,437 bp vs 18,583 bp in the Deadwood case), run KAT against the full read set to obtain each fraction's median k-mer abundance and kat-plot.pdf, and run Kraken 2 with Krona plots to compare taxonomic assignability (e.g. 56% vs 76% phylum-level bacteria, Legionella only in unmapped reads). Low abundance reads are expected to be enriched in the unmapped portion; similar lengths/qualities across fractions indicate abundance, not quality, drives unassembledness. Compile the metrics into a fraction-comparison summary table alongside read_contig_mapping.txt and report.txt.

## Related tools

- **Mapler** (Snakemake pipeline that aligns reads on contigs, computes mapped read/base ratios, and splits reads into mapped/unmapped FASTQ fractions) — https://github.com/Nimauric/Mapler
- **minimap2** (Aligns the HiFi reads on contigs to define mapped vs unmapped fractions)
- **FastQC** (Summarizes average read length and per-base quality of each read fraction) — https://github.com/s-andrews/FastQC
- **KAT** (Computes k-mer abundance statistics (kat-plot.pdf, <fraction_name>-stats.tsv) to show whether unmapped reads are dominated by abundance-1 k-mers) — https://github.com/EarlhamInst/KAT
- **Kraken2** (Taxonomic assignment of each read fraction for Krona-based comparison of assignability) — https://github.com/DerrickWood/kraken2
- **metaMDBG** (Assembler whose assembly output (assembly.fasta) serves as the contig reference for the read-mapping evaluation) — https://github.com/GaetanBenoitDev/metaMDBG

## Examples

```
./local_pipeline.sh config/config_deadwood.yaml > mylog.txt
```

## Evaluation signals

- Mapped vs unmapped average read lengths are similar (e.g. 18,437 bp vs 18,583 bp in the Deadwood reproduction), ruling out length as the assembly-exclusion cause
- Unmapped fraction's KAT distribution is dominated by k-mers of abundance 1 (median k-mer abundance of 1)
- Phylum-level assignable bacteria proportions reproduce the reported fraction gap (56% unmapped vs 76% mapped) within stochastic/deviation tolerance
- Taxa reported only in the unmapped fraction (e.g. Legionella species) are absent from the mapped fraction's Kraken 2 results
- Mapped read and base ratios in report.txt/read_contig_mapping.txt are consistent, with aligned read ratio not dramatically exceeding aligned base ratio (which would indicate only partial alignments)

## Limitations

- A mapped-read ratio significantly higher than the aligned base ratio means most reads only partially align to contigs, complicating fraction interpretation
- Fraction comparison is only run when both 'mapped' and 'unmapped' fractions are requested (kat-plot.pdf requires both)
- Taxonomic assignment requires external Kraken 2 (and GTDB-Tk for bins) databases to be configured; results depend on database choice and available disk space

## Evidence

- [readme] Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases, and separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment.: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases, and separately analyzes mapped and unmapped reads via their k-mer frequency"
- [methods] Typically, low abundance reads are more likely to be more abundant in the unmapped portions.: "Typically, low abundance reads are more libely to be more abundant in the unmapped portions"
- [methods] If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs.: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [methods] Taxonomic assignment of reads : done with kraken2: "Taxonomic assignment of reads : done with kraken2"
- [methods] separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment: "separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment"
