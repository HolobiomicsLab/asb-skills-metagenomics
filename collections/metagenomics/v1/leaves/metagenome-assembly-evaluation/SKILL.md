---
name: metagenome-assembly-evaluation
description: Use when HiFi long-read FASTQ reads and a metagenome assembly (FASTA,
  built-in or user-provided under outputs/<sample>/<assembler>/assembly.fasta) are
  available and you need to quantify how representative the assembly is of the sequenced
  sample.
when_to_use_negative:
- Only short-read (Illumina) data is available — Mapler is designed for HiFi long-read
  metagenomic assemblies (hybrid data would need OPERA-MS instead, not the primary
  evaluation route)
- You need de novo genome reconstruction benchmarks rather than assembly quality assessment
  — Mapler evaluates assemblies; it does not replace comparing binning-only workflows
- Input is raw reads with no assembly and no assembler enabled in the configfile —
  the evaluation rules require an assembly (built-in or user-submitted) to evaluate
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: Mapler
  role: Snakemake/Python/Bash pipeline orchestrating assembly and evaluation of HiFi
    metagenome assemblies
  repo: https://github.com/Nimauric/Mapler
- name: Snakemake
  role: workflow engine executing the Mapler evaluation rules (>=8.28, with SLURM
    executor plugin for cluster runs)
  repo: https://github.com/Nimauric/Mapler
- name: metaMDBG
  role: state-of-the-art HiFi metagenome assembler whose assemblies are evaluated;
    also usable via user-submitted assemblies
  repo: https://github.com/GaetanBenoitDev/metaMDBG
- name: minimap2
  role: aligns HiFi reads to assembly contigs to produce reads_on_contigs.bam and
    compute aligned read/base ratios
- name: CheckM2
  role: machine-learning bin quality assessment (quality_report.tsv) classifying MetaBAT2
    bins into quality categories
  repo: https://github.com/chklovski/CheckM2
- name: GTDB-Tk
  role: taxonomic assignment of bins used in classical bin quality classification
  repo: https://github.com/Ecogenomics/GTDBTk
- name: MetaQUAST
  role: 'assembly contiguity/misassembly evaluation enabled via metaquast: true'
  repo: https://github.com/ablab/quast
- name: hifiasm-meta
  role: alternative HiFi metagenome assembler available in Mapler's assemblers field
  repo: https://github.com/xfengnefx/hifiasm-meta
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:7ab148303a3faba2d8a8839cd959c84101826a6c8a22ba4eba967584b4df5a8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Bash
  - CheckM2
  - Conda
  - GTDB-Tk
  - Mapler
  - MetaQUAST
  - Python
  - Snakemake
  - hifiasm-meta
  - metaMDBG
  - metaflye
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# metagenome-assembly-evaluation

## Summary

Assess the quality of HiFi long-read metagenomic assemblies — either produced by Mapler's assemblers (metaMDBG, metaflye, hifiasm-meta) or user-submitted — using aligned read/base ratios, bin quality classification (CheckM2, GTDB-Tk), and k-mer/taxonomic analysis of mapped vs unmapped reads.

## When to use

Use when HiFi long-read FASTQ reads and a metagenome assembly (FASTA, built-in or user-provided under outputs/<sample>/<assembler>/assembly.fasta) are available and you need to quantify how representative the assembly is of the sequenced sample.

## When NOT to use

- Only short-read (Illumina) data is available — Mapler is designed for HiFi long-read metagenomic assemblies (hybrid data would need OPERA-MS instead, not the primary evaluation route)
- You need de novo genome reconstruction benchmarks rather than assembly quality assessment — Mapler evaluates assemblies; it does not replace comparing binning-only workflows
- Input is raw reads with no assembly and no assembler enabled in the configfile — the evaluation rules require an assembly (built-in or user-submitted) to evaluate

## Inputs

- HiFi long-read FASTQ files (read_path entries in the configfile)
- Assembly FASTA files (built-in metaMDBG/metaflye/hifiasm-meta output or user-provided at outputs/<sample>/<custom_assembly_process>/assembly.fasta)
- Optional user-provided bins (FASTA under outputs/<sample>/<assembly>/<custom_binning_process>_bins_reads_alignement/bins/)
- config.yaml derived from config/config_template.yaml (inputs, controls, parameters sections)
- External databases for taxonomic assignment (Kraken 2 database, GTDB-Tk reference data) when those analyses are enabled

## Outputs

- outputs/<sample_name>/<assembler>/assembly.fasta
- reads_on_contigs_mapping_evaluation/report.txt (aligned reads and aligned bases ratios)
- metabat2_bins_reads_alignement/checkm/checkm_report.txt (near complete, high, medium, low quality bin counts)
- read_contig_mapping.txt and read_contig_mapping_plot.pdf (aligned ratios split by bin quality class)
- metaquast, kraken2, kat, fastqc reports and plots under outputs/<sample_name>/<assembler>/ depending on enabled controls
- Logs per analysis in logs/<analysis_name>/<date_hour>

## How to apply

Copy config/config_template.yaml to a configfile listing sample read_path entries and enable the desired assemblers plus evaluation controls (e.g. read_mapping_evaluation: true, binning: true with checkm: true, metaquast, kraken2). Run the pipeline via Snakemake ('./local_pipeline.sh <configfile>' locally or 'sbatch pipeline.sh <configfile>' on SLURM) with configurable resource parameters (threads, memory, time). Core evaluation aligns HiFi reads to contigs with minimap2 (reads_on_contigs.bam), computes aligned read ratio (aligned read count / total reads) and aligned base ratio (aligned bases / total read length) in reads_on_contigs_mapping_evaluation/report.txt; a higher aligned read ratio than base ratio signals mostly partially aligned reads. When binning is enabled, MetaBAT2 bins are classified by CheckM2 into near complete, high, medium and low quality, and per-quality-class ratios are reported in read_contig_mapping.txt and read_contig_mapping_plot.pdf, with multi-contig reads counted only at the highest bin quality level touched. Validate by confirming assembly.fasta and reads_on_contigs_mapping_evaluation/report.txt exist and that ratios fall in [0, 1], with higher ratios indicating a more representative assembly.

## Related tools

- **Mapler** (Snakemake/Python/Bash pipeline orchestrating assembly and evaluation of HiFi metagenome assemblies) — https://github.com/Nimauric/Mapler
- **Snakemake** (workflow engine executing the Mapler evaluation rules (>=8.28, with SLURM executor plugin for cluster runs)) — https://github.com/Nimauric/Mapler
- **metaMDBG** (state-of-the-art HiFi metagenome assembler whose assemblies are evaluated; also usable via user-submitted assemblies) — https://github.com/GaetanBenoitDev/metaMDBG
- **minimap2** (aligns HiFi reads to assembly contigs to produce reads_on_contigs.bam and compute aligned read/base ratios)
- **CheckM2** (machine-learning bin quality assessment (quality_report.tsv) classifying MetaBAT2 bins into quality categories) — https://github.com/chklovski/CheckM2
- **GTDB-Tk** (taxonomic assignment of bins used in classical bin quality classification) — https://github.com/Ecogenomics/GTDBTk
- **MetaQUAST** (assembly contiguity/misassembly evaluation enabled via metaquast: true) — https://github.com/ablab/quast
- **hifiasm-meta** (alternative HiFi metagenome assembler available in Mapler's assemblers field) — https://github.com/xfengnefx/hifiasm-meta

## Examples

```
conda create -n mapler -c bioconda -c conda-forge 'bioconda::snakemake>=8.28' 'conda-forge::conda>=24.1.2' bioconda::snakemake-executor-plugin-slurm && ./local_pipeline.sh config/config_test.yaml > mylog.txt
```

## Evaluation signals

- outputs/<sample_name>/<assembler>/assembly.fasta and reads_on_contigs_mapping_evaluation/report.txt are present, confirming the run completed
- Aligned read and aligned base ratios in report.txt fall within [0, 1], with higher ratios indicating a more representative assembly
- A significantly higher aligned read ratio than aligned base ratio correctly flags mostly partially aligned reads rather than an execution error
- read_contig_mapping.txt exists and reports ratios split by near complete / high / medium / low quality bins (and unbinned contigs), with read_contig_mapping_plot.pdf as its visual counterpart
- Reference reproduction: evaluation of three assemblers at 48 CPUs / 200G completed in roughly 2 h 4 min (Mock community), 3 h 46 min (Gut microbiome), 5 h 3 min (Deadwood); test dataset runs in ~15 minutes with 16 CPUs and 15G RAM

## Limitations

- Aligned ratios decrease with dataset complexity: in taxonomically rich environmental samples (e.g. Deadwood with hundreds to thousands of populations), only ~12.6% of reads and ~9.4% of bases aligned to medium-quality-or-higher bins in the metaMDBG assembly
- Low abundance reads are more likely to be abundant in the unmapped portions, so low aligned ratios can reflect genuine uncaptured diversity rather than assembly failure
- Optional taxonomic analyses require external databases (Kraken 2, GTDB-Tk) whose paths must be set in the configfile; resource parameters (threads, memory, time) must be tuned per rule, and a crashed run requires 'snakemake --unlock' before relaunching

## Evidence

- [readme] The aim of this tool is to evaluate HiFi long-reads metagenomic assemblies, and it can either perform the assembly itself using multiple state-of-the-art assemblers: "The aim of this tool is to evaluate HiFi long-reads metagenomic assemblies, and it can either perform the assembly itself using multiple state-of-the-art assemblers (metaMDBG, metaflye, hifiasm-meta)"
- [readme] Mapler analyzes the alignment of reads on contigs: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
- [methods] The higher those ratios are, the more representative of the sequenced sample is the assembly.: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [readme] with 16 CPUs and 15G of RAM, it should require around 15 minutes: "Either way, with 16 CPUs and 15G of RAM, it should require around 15 minutes, not including dependency installation, to produce the following results"
- [abstract] It incorporates several state-of-the-art metrics, as well as novel metrics assessing the diversity that remains uncaptured by the assembly process.: "It incorporates several state-of-the-art metrics, as well as novel metrics assessing the diversity that remains uncaptured by the assembly process."
