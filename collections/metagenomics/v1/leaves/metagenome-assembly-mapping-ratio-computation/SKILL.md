---
name: metagenome-assembly-mapping-ratio-computation
description: Use when evaluating a HiFi long-read metagenome assembly by quantifying
  the proportion of raw reads and read bases that align to contigs, optionally stratified
  by bin quality (e.g. medium quality or higher).
when_to_use_negative:
- The goal is assembly reconstruction itself rather than evaluation — Mapler's evaluation
  ratio is an assessment metric, not an assembly step
- Only short-read data is available — the pipeline and its benchmarks target HiFi
  long-read metagenomic assemblies
- A bin-quality-stratified mapping report is needed but binning/checkm cannot be enabled
  (bins absent), in which case only the unstratified report.txt can be produced
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Mapler
  role: Snakemake pipeline orchestrating the read-on-contig alignment, mapping ratio
    computation, and quality-stratified reporting
  repo: https://github.com/Nimauric/Mapler
- name: Snakemake
  role: Workflow engine executing Mapler rules (reads_on_contigs, reads_on_contigs_mapping_evaluation,
    read_contig_mapping) with Conda environments
- name: minimap2
  role: Aligns HiFi reads on assembly contigs to produce reads_on_contigs.bam used
    for ratio computation
- name: CheckM2
  role: Predicts bin completeness and contamination so mapping ratios can be split
    by bin quality class
  repo: https://github.com/chklovski/CheckM2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:3db072817874367a0e689d80a2d008bc1f70fb248239a52ac2127fc374349388
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Bash
  - CheckM2
  - Conda
  - Mapler
  - Python
  - Snakemake
  - metaMDBG
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metagenome-assembly-mapping-ratio-computation

## Summary

Computes aligned read and aligned base ratios of HiFi reads on metagenome assembly contigs, optionally split by bin quality class, to quantify how representative an assembly is of the sequenced sample. Part of the Mapler (MAEPLR) Snakemake pipeline.

## When to use

Use when evaluating a HiFi long-read metagenome assembly by quantifying the proportion of raw reads and read bases that align to contigs, optionally stratified by bin quality (e.g. medium quality or higher).

## When NOT to use

- The goal is assembly reconstruction itself rather than evaluation — Mapler's evaluation ratio is an assessment metric, not an assembly step
- Only short-read data is available — the pipeline and its benchmarks target HiFi long-read metagenomic assemblies
- A bin-quality-stratified mapping report is needed but binning/checkm cannot be enabled (bins absent), in which case only the unstratified report.txt can be produced

## Inputs

- Raw HiFi read FASTQ files (path declared in the configfile read_path entries)
- Assembly FASTA placed at outputs/<sample_name>/<custom_assembly_process>/assembly.fasta
- Mapler configfile (copy of config/config_template.yaml) with read_mapping_evaluation: true and optionally binning: true / checkm: true

## Outputs

- reads_on_contigs.bam and reads_on_contigs.bam.bai (minimap2 alignment of reads on contigs)
- reads_on_contigs_mapping_evaluation/report.txt (overall aligned read ratio and aligned base ratio)
- read_contig_mapping.txt (aligned read and base ratios split by bin quality class)
- read_contig_mapping_plot.pdf (visual version of read_contig_mapping.txt)

## How to apply

Place the assembly at outputs/<sample_name>/<custom_assembly_process>/assembly.fasta and declare raw HiFi FASTQ files via read_path entries in a copy of config/config_template.yaml. Enable read_mapping_evaluation: true and, for quality-stratified ratios, binning: true with checkm: true, running through Snakemake/Conda (environment 'mapler' with bioconda::snakemake>=8.28). The reads_on_contigs step aligns HiFi reads to contigs with minimap2 to produce reads_on_contigs.bam and reads_on_contigs.bam.bai, then the reads_on_contigs_mapping_evaluation rule computes the aligned read ratio (aligned read count / total read count) and the aligned base ratio (aligned read bases / total read length) into report.txt. With binning enabled, the CheckM2-backed read_contig_mapping step splits the same ratios by bin quality class (near complete, high quality, medium quality, low quality) into read_contig_mapping.txt and read_contig_mapping_plot.pdf. Interpretation follows the paper: higher ratios mean a more representative assembly, and an aligned read ratio significantly above the aligned base ratio signals that most reads are only partially aligned to contigs.

## Related tools

- **Mapler** (Snakemake pipeline orchestrating the read-on-contig alignment, mapping ratio computation, and quality-stratified reporting) — https://github.com/Nimauric/Mapler
- **Snakemake** (Workflow engine executing Mapler rules (reads_on_contigs, reads_on_contigs_mapping_evaluation, read_contig_mapping) with Conda environments)
- **minimap2** (Aligns HiFi reads on assembly contigs to produce reads_on_contigs.bam used for ratio computation)
- **CheckM2** (Predicts bin completeness and contamination so mapping ratios can be split by bin quality class) — https://github.com/chklovski/CheckM2

## Examples

```
./local_pipeline.sh config/config_template.yaml > mylog.txt
```

## Evaluation signals

- reads_on_contigs_mapping_evaluation/report.txt exists and contains both the aligned read ratio and aligned base ratio
- Aligned read ratio equals aligned read count / total read count and aligned base ratio equals aligned read bases / total read length, reproducible on re-run
- Published reference ratios reproduced within tolerance: 96.9%/96.1% (Mock community), 49.5%/47.8% (Gut microbiome), 12.6%/9.4% at medium quality or higher (Deadwood, metaMDBG)
- With binning enabled, read_contig_mapping.txt reports ratios split across the four quality classes (near complete, high quality, medium quality, low quality)
- A large gap between aligned read ratio and aligned base ratio is checked as an indicator of partially aligned reads

## Limitations

- Low abundance reads are typically more abundant in the unmapped portion, so low mapping ratios for complex samples partly reflect uncaptured low-abundance diversity rather than only assembly failure
- The aligned read ratio versus aligned base ratio gap indicates partially aligned reads; the ratio alone does not distinguish full from partial alignment
- Bin-quality-stratified ratios require binning (MetaBAT2) and quality assessment (CheckM2) plus optional external databases (Kraken 2, GTDB-Tk) whose paths must be configured

## Evidence

- [methods] Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
- [methods] Higher ratios indicate more representative assembly: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] Aligned read ratio above base ratio indicates partial alignment: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [readme] README: reads_on_contigs_mapping_evaluation rule outputs report.txt on aligned reads and bases: "└── reads_on_contigs_mapping_evaluation
        └── report.txt # A report on the aligned reads and aligned bases ratios"
- [readme] README: read_contig_mapping.txt reports aligned ratios split by bin quality: "└── read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
