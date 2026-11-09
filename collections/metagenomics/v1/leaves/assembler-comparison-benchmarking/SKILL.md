---
name: assembler-comparison-benchmarking
description: Use when comparing metaMDBG, metaflye and hifiasm-meta on the same HiFi
  read set of a taxonomically rich metagenome to quantify uncaptured diversity (unaligned
  bases) and bases aligned to medium-or-better bins.
when_to_use_negative:
- The input is a single user-provided assembly with no alternate assembler to compare
  — Mapler's evaluation-only mode (config_test_evaluation_only.yaml) is the right
  choice instead
- The reads are short reads or non-HiFi nanopore data; Mapler focuses on evaluating
  HiFi long-read metagenomic assemblies
- You only want assembly statistics (contiguity, N50) rather than uncaptured-diversity
  metrics based on read-to-contig alignment
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Mapler
  role: Snakemake pipeline that runs the assemblers and computes read-mapping evaluation
    metrics for comparison
  repo: https://github.com/Nimauric/Mapler
- name: metaMDBG
  role: Built-in assembler option; on Deadwood it left 52.7% of bases unaligned, the
    best captured diversity
  repo: https://github.com/GaetanBenoitDev/metaMDBG
- name: metaflye
  role: Built-in assembler option; recovered the most bases but left 76.7% of bases
    unaligned
  repo: https://github.com/mikolmogorov/Flye
- name: hifiasm-meta
  role: Built-in assembler option; 66.1% unaligned bases and the highest fraction
    of bases aligned to medium-or-better bins (14.2% vs 12.6%)
  repo: https://github.com/xfengnefx/hifiasm-meta
- name: MetaBAT2
  role: Bins each assembly's contigs so mapping metrics can be split by bin quality
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/
- name: CheckM2
  role: Classifies bins into near-complete, high, medium and low quality categories
    for the stratified comparison
  repo: https://github.com/chklovski/CheckM2
- name: minimap2
  role: Aligns reads on each assembly's contigs to produce the aligned read/base ratios
- name: snakemake-executor-plugin-slurm
  role: Enables sbatch pipeline.sh cluster execution of the comparison workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:1cc20a6685a015f3b2478948040b0a6680462323ff8c239b89e85a849aa5695c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - CheckM2
  - Conda
  - Mapler
  - MetaBAT2
  - hifiasm-meta
  - metaMDBG
  - metaflye
  - minimap2
  - snakemake-executor-plugin-slurm
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# assembler-comparison-benchmarking

## Summary

Benchmark multiple HiFi metagenome assemblers (metaMDBG, metaflye, hifiasm-meta) on the same read set using the Mapler Snakemake pipeline, comparing the diversity left uncaptured by assembly through aligned read/base ratios and per-bin-quality mapping breakdowns.

## When to use

Use when comparing metaMDBG, metaflye and hifiasm-meta on the same HiFi read set of a taxonomically rich metagenome to quantify uncaptured diversity (unaligned bases) and bases aligned to medium-or-better bins.

## When NOT to use

- The input is a single user-provided assembly with no alternate assembler to compare — Mapler's evaluation-only mode (config_test_evaluation_only.yaml) is the right choice instead
- The reads are short reads or non-HiFi nanopore data; Mapler focuses on evaluating HiFi long-read metagenomic assemblies
- You only want assembly statistics (contiguity, N50) rather than uncaptured-diversity metrics based on read-to-contig alignment

## Inputs

- HiFi long-read FASTQ sample (e.g. Deadwood co-assembly reads from doi:10.57745/2SA8AB)
- Mapler YAML configfile listing assemblers (metaMDBG, metaflye, hifiasm_meta), binners, and control flags (binning, checkm, read_mapping_evaluation)

## Outputs

- outputs/<sample>/<assembler>/assembly.fasta for each assembler
- metabat2_bins_reads_alignement/checkm/checkm_report.txt with near-complete, high, medium and low quality bin counts
- reads_on_contigs_mapping_evaluation/report.txt with aligned read ratio and aligned base ratio
- read_contig_mapping.txt / read_contig_mapping_plot.pdf with mapping split by bin quality
- assembler_comparison_mapler.csv compiling unaligned base percentages and medium-quality-bin aligned bases per assembler

## How to apply

Retrieve the HiFi read set (e.g. the Deadwood co-assembly reads from doi:10.57745/2SA8AB) and write a Mapler configfile (copied from config/config_template.yaml) listing metaMDBG, metaflye and hifiasm_meta in the assemblers field with binning: true, checkm: true and read_mapping_evaluation: true. Launch the pipeline with ./local_pipeline.sh <configfile> > mylog.txt locally or sbatch pipeline.sh <configfile> on SLURM, so each assembler produces outputs/<sample>/<assembler>/assembly.fasta. Bin each assembly with MetaBAT2 and classify bins into near-complete, high, medium and low quality categories from the CheckM report. For each assembler, read reads_on_contigs_mapping_evaluation/report.txt to compute the aligned read ratio (aligned read count / total read count) and aligned base ratio (aligned read bases / total read length) — higher ratios mean the assembly better represents the sample, and an aligned read ratio far above the base ratio signals mostly partially aligned reads. Finally, extract the per-bin-quality mapping breakdown from read_contig_mapping.txt / read_contig_mapping_plot.pdf and compile the unaligned base percentages and medium-quality-bin-aligned percentages into a comparison table (e.g. assembler_comparison_mapler.csv).

## Related tools

- **Mapler** (Snakemake pipeline that runs the assemblers and computes read-mapping evaluation metrics for comparison) — https://github.com/Nimauric/Mapler
- **metaMDBG** (Built-in assembler option; on Deadwood it left 52.7% of bases unaligned, the best captured diversity) — https://github.com/GaetanBenoitDev/metaMDBG
- **metaflye** (Built-in assembler option; recovered the most bases but left 76.7% of bases unaligned) — https://github.com/mikolmogorov/Flye
- **hifiasm-meta** (Built-in assembler option; 66.1% unaligned bases and the highest fraction of bases aligned to medium-or-better bins (14.2% vs 12.6%)) — https://github.com/xfengnefx/hifiasm-meta
- **MetaBAT2** (Bins each assembly's contigs so mapping metrics can be split by bin quality) — https://bitbucket.org/berkeleylab/metabat/src/master/
- **CheckM2** (Classifies bins into near-complete, high, medium and low quality categories for the stratified comparison) — https://github.com/chklovski/CheckM2
- **minimap2** (Aligns reads on each assembly's contigs to produce the aligned read/base ratios)
- **snakemake-executor-plugin-slurm** (Enables sbatch pipeline.sh cluster execution of the comparison workflow)

## Examples

```
sbatch pipeline.sh config/config_deadwood_assemblers.yaml
```

## Evaluation signals

- Per-assembler unaligned base percentages reproduce the reference values: 52.7% (metaMDBG), 76.7% (metaflye), 66.1% (hifiasm-meta) on the Deadwood co-assembly
- Bases aligned to at least medium-quality bins match the reported 14.2% (hifiasm-meta) vs 12.6% (metaMDBG), confirming hifiasm-meta captures more medium-quality-bin bases despite a higher unaligned fraction
- Each assembler directory contains outputs/<sample>/<assembler>/assembly.fasta plus metabat2_bins_reads_alignement and reads_on_contigs_mapping_evaluation subfolders
- reads_on_contigs_mapping_evaluation/report.txt reports both an aligned read ratio and an aligned base ratio; a read ratio significantly higher than the base ratio flags mostly partially aligned reads
- checkm_report.txt contains a count of near-complete, high quality, medium quality and low quality bins followed by per-bin details

## Limitations

- Custom assembly processes cannot share names with built-in assemblers (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS), and custom binners cannot be named metabat2 or custom
- The test config (config_test.yaml) runs only part of the analysis on the test dataset, so full assembler comparisons require user-specified configfiles and databases
- Low abundance reads are more likely to be abundant in the unmapped portion, so unaligned base percentages partly reflect low-abundance populations rather than assembler failure alone
- Taxonomic assignment of reads and bins requires external Kraken 2 and GTDB-Tk databases whose paths must be set in the configfile

## Evidence

- [abstract] Mapler facilitates the comparison of assembly strategies and helps identify methodological bottlenecks that hinder genome reconstruction.: "Mapler facilitates the comparison of assembly strategies and helps identify methodological bottlenecks that hinder genome reconstruction."
- [methods] Higher ratios indicate more representative assembly: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [readme] Pipeline launch commands: "./local_pipeline.sh <configfile> > mylog.txt # for local execution"
- [readme] Per-bin-quality mapping report: "read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
- [methods] Mapler mapping ratio computation: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
