---
name: quality-stratified-mapping-visualization
description: Use when evaluating a HiFi metagenome assembly where bins have CheckM2
  quality scores, and you need mapping ratios resolved to the highest bin quality
  level each read touches.
when_to_use_negative:
- No binning has been performed — use the non-binning-aware mapping evaluation (reads_on_contigs_mapping_evaluation/report.txt)
  instead
- Assemblies from short reads or non-HiFi data — Mapler's evaluation is focused on
  HiFi long-read metagenome assemblies
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: minimap2
  role: Aligns HiFi long reads to assembly contigs, producing the sorted BAM used
    to compute mapping ratios
- name: CheckM2
  role: Provides quality_report.tsv used to classify bins into quality categories
    for stratification
  repo: https://github.com/chklovski/CheckM2
- name: Python
  role: pysam reads the BAM, pandas aggregates per-category ratios into the text report,
    matplotlib renders the PDF plot
- name: Mapler
  role: Snakemake pipeline that orchestrates the binning-aware mapping evaluation
    and report/plot generation
  repo: https://github.com/Nimauric/Mapler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:90df7ab2f4c3cbda3e0e1a8668218440d6d5a22e4ca4bcf2e751782a5893d64f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chklovski/CheckM2
  tools:
  - CheckM2
  - Mapler
  - Python
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Quality-stratified mapping visualization

## Summary

Computes Mapler's binning-aware aligned read and aligned base percentage metrics, splitting read-to-contig alignments by bin quality category (near complete, high, medium, low quality, unbinned) and rendering them both as a text report and a PDF plot.

## When to use

Use when evaluating a HiFi metagenome assembly where bins have CheckM2 quality scores, and you need mapping ratios resolved to the highest bin quality level each read touches.

## When NOT to use

- No binning has been performed — use the non-binning-aware mapping evaluation (reads_on_contigs_mapping_evaluation/report.txt) instead
- Assemblies from short reads or non-HiFi data — Mapler's evaluation is focused on HiFi long-read metagenome assemblies

## Inputs

- HiFi long reads (FASTQ)
- Assembly contigs (assembly.fasta)
- Sorted BAM of reads aligned to contigs (reads_on_contigs.bam with .bai)
- CheckM2 quality_report.tsv (bin completeness/contamination)

## Outputs

- read_contig_mapping.txt — aligned read and aligned base ratios split by bin quality category
- read_contig_mapping_plot.pdf — visual version of read_contig_mapping.txt

## How to apply

Align the HiFi long reads to the assembly contigs with Minimap2 and produce a sorted BAM index (reads_on_contigs.bam with .bai) handled via pysam. Parse the CheckM2 quality_report.tsv to classify each bin into near complete, high quality, medium quality or low quality categories, as summarized in checkm_report.txt. Assign every contig to its bin's quality category (unbinned contigs form their own category) and, for reads mapping multiple contigs, count each read only once at the highest bin quality level it touches. Per category compute the aligned read ratio (aligned reads / total reads) and the aligned base ratio (aligned read bases / total read length), aggregate them with pandas into read_contig_mapping.txt, and render the same breakdown with matplotlib as read_contig_mapping_plot.pdf. Higher ratios indicate the assembly is more representative of the sequenced sample; a read ratio far above the base ratio signals mostly partially aligned reads.

## Related tools

- **minimap2** (Aligns HiFi long reads to assembly contigs, producing the sorted BAM used to compute mapping ratios)
- **CheckM2** (Provides quality_report.tsv used to classify bins into quality categories for stratification) — https://github.com/chklovski/CheckM2
- **Python** (pysam reads the BAM, pandas aggregates per-category ratios into the text report, matplotlib renders the PDF plot)
- **Mapler** (Snakemake pipeline that orchestrates the binning-aware mapping evaluation and report/plot generation) — https://github.com/Nimauric/Mapler

## Examples

```
snakemake --use-conda -c1 --configfile config/config_test.yaml && cat outputs/test_dataset/metaMDBG/metabat2_bins_reads_alignement/read_contig_mapping.txt
```

## Evaluation signals

- read_contig_mapping.txt exists and reports aligned read and aligned base ratios split by bin quality (near complete, high, medium, low, unbinned)
- read_contig_mapping_plot.pdf is generated as the visual counterpart of read_contig_mapping.txt
- All reported ratios fall within [0, 1]
- Higher aligned ratios indicate a more representative assembly; aligned read ratio significantly above aligned base ratio signals partially aligned reads
- Multi-contig reads are counted only once, at the highest bin quality level they touch

## Limitations

- Ratios depend on binning and CheckM2 quality estimates; reads touching only unbinned contigs fall into the unbinned category
- Typically low abundance reads are more likely to be abundant in the unmapped portions, so low ratios may reflect sample composition rather than assembly failure
- Full stratified analysis requires binning (MetaBAT2) and CheckM2 to have been run; only part of the analysis runs on the test dataset

## Evidence

- [methods] Binning-aware mapping ratio computation: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
- [methods] Higher ratios indicate representativeness: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] Partially aligned reads diagnostic: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [readme] Output artifacts of the binning-aware evaluation: "read_contig_mapping_plot.pdf # A visual version of read_contig_mapping.txt
read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
- [readme] Bin quality categories from marker gene content: "In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment, Mapler analyzes the alignment of reads on contigs."
