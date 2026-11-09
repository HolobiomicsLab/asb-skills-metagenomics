---
name: bin-quality-categorization-from-checkm
description: Use when evaluating a HiFi long-read metagenome assembly that has been
  binned (e.g. MetaBAT2 bins scored by CheckM2) and you need read/base mapping ratios
  split by bin quality category instead of a single aggregate alignment ratio.
when_to_use_negative:
- You only need a single global aligned-read/aligned-base ratio without binning-aware
  breakdown (Mapler's plain reads_on_contigs_mapping_evaluation/report.txt covers
  that).
- No binning results or CheckM2 quality scores are available, so contigs cannot be
  assigned to a bin quality category.
- Input assemblies are short-read or isolate assemblies rather than HiFi long-read
  metagenome assemblies.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3941
tools:
- name: Mapler
  role: Pipeline hosting the binning-aware read/contig mapping evaluation module;
    implemented as Snakemake rules producing read_contig_mapping.txt and read_contig_mapping_plot.pdf
  repo: https://github.com/Nimauric/Mapler
- name: CheckM2
  role: Provides quality_report.tsv (completeness and contamination) used to classify
    bins into quality categories
  repo: https://github.com/chklovski/CheckM2
- name: minimap2
  role: Aligns HiFi long reads to assembly contigs to produce the BAM used for per-category
    ratio computation
- name: Python
  role: Implements the module with pysam (BAM handling), pandas (report aggregation)
    and matplotlib (PDF plotting)
  repo: https://github.com/Nimauric/Mapler
- name: Snakemake
  role: Workflow engine that orchestrates the alignment, bin classification and report
    generation steps
  repo: https://github.com/Nimauric/Mapler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:410a9fe487216c315ad305ac1253a1e78559a1a7157cea3265675d028eedbcd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - CheckM2
  - Mapler
  - Python
  - Snakemake
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bin-quality-categorization-from-checkm

## Summary

A binning-aware evaluation module of the Mapler pipeline that classifies assembly bins into quality categories from a CheckM2 quality_report.tsv and computes aligned read and aligned base percentage metrics per category, using HiFi read-to-contig alignments. It exposes assembly representativeness stratified by genome bin quality rather than as a single global ratio.

## When to use

Use when evaluating a HiFi long-read metagenome assembly that has been binned (e.g. MetaBAT2 bins scored by CheckM2) and you need read/base mapping ratios split by bin quality category instead of a single aggregate alignment ratio.

## When NOT to use

- You only need a single global aligned-read/aligned-base ratio without binning-aware breakdown (Mapler's plain reads_on_contigs_mapping_evaluation/report.txt covers that).
- No binning results or CheckM2 quality scores are available, so contigs cannot be assigned to a bin quality category.
- Input assemblies are short-read or isolate assemblies rather than HiFi long-read metagenome assemblies.

## Inputs

- HiFi long reads in FASTQ/FASTA format
- Metagenome assembly contigs (assembly.fasta)
- Bin FASTA files (bins/*.fa, e.g. MetaBAT2 output)
- CheckM2 quality_report.tsv with per-bin completeness and contamination
- Sorted read-to-contig alignment BAM (reads_on_contigs.bam with .bai) from Minimap2

## Outputs

- read_contig_mapping.txt: text report of aligned read and aligned base ratios split by bin quality category
- read_contig_mapping_plot.pdf: graphical (matplotlib) version of read_contig_mapping.txt
- checkm_report.txt: count of near complete, high quality, medium quality and low quality bins plus per-bin details

## How to apply

Align the HiFi long reads to the assembly contigs with Minimap2 and produce a sorted BAM plus index (reads_on_contigs.bam with .bai) via pysam. Parse the CheckM2 quality_report.tsv to classify each bin into the near complete, high quality, medium quality or low quality categories as reported in checkm_report.txt. Assign every contig to its bin's quality category (unbinned contigs form their own category) and, for reads mapping multiple contigs, resolve each read to the highest bin quality level it touches. Per category, compute the aligned read ratio (aligned read count / total read count) and the aligned base ratio (aligned read bases / total read length), aggregate them with pandas into the text report read_contig_mapping.txt, and render the same breakdown with matplotlib as read_contig_mapping_plot.pdf. Higher ratios indicate an assembly more representative of the sequenced sample; an aligned read ratio significantly above the aligned base ratio signals that most reads are only partially aligned.

## Related tools

- **Mapler** (Pipeline hosting the binning-aware read/contig mapping evaluation module; implemented as Snakemake rules producing read_contig_mapping.txt and read_contig_mapping_plot.pdf) — https://github.com/Nimauric/Mapler
- **CheckM2** (Provides quality_report.tsv (completeness and contamination) used to classify bins into quality categories) — https://github.com/chklovski/CheckM2
- **minimap2** (Aligns HiFi long reads to assembly contigs to produce the BAM used for per-category ratio computation)
- **Python** (Implements the module with pysam (BAM handling), pandas (report aggregation) and matplotlib (PDF plotting)) — https://github.com/Nimauric/Mapler
- **Snakemake** (Workflow engine that orchestrates the alignment, bin classification and report generation steps) — https://github.com/Nimauric/Mapler

## Examples

```
./local_pipeline.sh config/config_test_evaluation_only.yaml > mylog.txt
```

## Evaluation signals

- read_contig_mapping.txt exists and reports aligned read and aligned base ratios split by bin quality (near complete, high, medium, low, unbinned)
- read_contig_mapping_plot.pdf is generated as the visual counterpart of read_contig_mapping.txt
- All reported ratios fall within [0, 1]
- Per-category counts respect the rule that multi-contig reads are counted only at the highest bin quality level they touch
- A significantly higher aligned read ratio than aligned base ratio is interpreted as mostly partially aligned reads, consistent with the pipeline's documented interpretation

## Limitations

- Low abundance reads are more likely to be abundant in the unmapped portion, so unmapped categories can be dominated by low-abundance organisms
- The binning-aware breakdown depends on upstream binning (e.g. MetaBAT2) and CheckM2 quality estimates; poor binning or misclassification propagates into the categories
- Running the full evaluation requires Snakemake and Conda (plus the SLURM executor plugin on clusters); only part of the analysis runs on the bundled test dataset

## Evidence

- [readme] In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment, Mapler analyzes the alignment of reads on contigs.: "In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment, Mapler analyzes the alignment of reads on contigs."
- [readme] Mapler analyzes the alignment of reads on contigs by calculating the ratio of mapped reads and bases.: "It does this by calculating the ratio of mapped reads and bases, and separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment."
- [methods] The higher those ratios are, the more representative of the sequenced sample is the assembly.: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs.: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [readme] read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality: "read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
- [readme] checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins: "checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins"
