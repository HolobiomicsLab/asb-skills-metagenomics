---
name: read-mapping-ratio-computation
description: Use when evaluating a metagenome assembly by measuring what fraction
  of HiFi reads and read bases align to contigs, especially when the breakdown should
  be stratified by bin quality (near complete/high/medium/low/unbinned).
when_to_use_negative:
- The goal is assembler-level contiguity or completeness statistics (e.g. N50, MetaQUAST
  metrics) rather than how much of the read set the assembly represents.
- No assembly contigs are available yet — this skill evaluates an existing assembly
  and cannot be run pre-assembly.
- Reads are short reads rather than HiFi long reads, since Mapler's evaluation logic
  and tooling are designed for HiFi long-read metagenomic assemblies.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Mapler
  role: Pipeline orchestrating the read-to-contig alignment and mapping ratio computation,
    reporting both overall and binning-aware breakdowns
  repo: https://github.com/Nimauric/Mapler
- name: minimap2
  role: Aligns the HiFi long reads onto the assembly contigs, producing the sorted
    BAM index used for ratio computation
- name: CheckM2
  role: Supplies quality_report.tsv used to classify bins into near complete, high,
    medium and low quality categories for the binning-aware breakdown
  repo: https://github.com/chklovski/CheckM2
- name: Snakemake
  role: Workflow engine executing the read-mapping evaluation steps within Mapler
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:d7099b390b7daf8e1747c925b1659be1a851383c67bf76daec2fb4a90b09c0a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - CheckM2
  - Conda
  - Mapler
  - MetaBAT2
  - Python
  - Snakemake
  - hifiasm-meta
  - metaMDBG
  - metaflye
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# read-mapping-ratio-computation

## Summary

Compute aligned read and aligned base ratios by aligning HiFi long reads to assembly contigs, optionally broken down by bin quality category, to quantify how much of the sequenced sample's diversity the assembly captures.

## When to use

Use when evaluating a metagenome assembly by measuring what fraction of HiFi reads and read bases align to contigs, especially when the breakdown should be stratified by bin quality (near complete/high/medium/low/unbinned).

## When NOT to use

- The goal is assembler-level contiguity or completeness statistics (e.g. N50, MetaQUAST metrics) rather than how much of the read set the assembly represents.
- No assembly contigs are available yet — this skill evaluates an existing assembly and cannot be run pre-assembly.
- Reads are short reads rather than HiFi long reads, since Mapler's evaluation logic and tooling are designed for HiFi long-read metagenomic assemblies.

## Inputs

- HiFi long-read FASTQ read set (e.g. Deadwood reads from doi:10.57745/2SA8AB)
- Assembly contigs (assembly.fasta) from metaMDBG, metaflye, hifiasm-meta or a user-submitted assembly
- CheckM2 quality_report.tsv for bin quality classification (when binning: true)
- Bin FASTA files from MetaBAT2 (when binning: true)

## Outputs

- reads_on_contigs.bam (+ .bai): sorted Minimap2 read-to-contig alignments
- reads_on_contigs_mapping_evaluation/report.txt: overall aligned read and aligned base ratios
- read_contig_mapping.txt: aligned read/base ratios split by bin quality category
- read_contig_mapping_plot.pdf: visual counterpart of read_contig_mapping.txt

## How to apply

Align the HiFi long reads to the assembly contigs with Minimap2, producing a sorted, indexed BAM (reads_on_contigs.bam with .bai) handled via pysam. If binning was performed, take the CheckM2 quality_report.tsv to classify each bin into near complete, high quality, medium quality or low quality categories (as reported in checkm_report.txt), assign every contig to its bin's quality category with unbinned contigs forming their own category, and resolve reads mapping multiple contigs to the highest bin quality level they touch. Per category, compute the aligned read ratio (aligned read count / total read count) and the aligned base ratio (aligned read bases / total read length), then aggregate the per-category ratios with pandas into read_contig_mapping.txt and render the same breakdown graphically with matplotlib as read_contig_mapping_plot.pdf. Rationale: the higher these ratios are, the more representative of the sequenced sample the assembly is; a gap between read and base ratios signals partially aligned reads.

## Related tools

- **Mapler** (Pipeline orchestrating the read-to-contig alignment and mapping ratio computation, reporting both overall and binning-aware breakdowns) — https://github.com/Nimauric/Mapler
- **minimap2** (Aligns the HiFi long reads onto the assembly contigs, producing the sorted BAM index used for ratio computation)
- **CheckM2** (Supplies quality_report.tsv used to classify bins into near complete, high, medium and low quality categories for the binning-aware breakdown) — https://github.com/chklovski/CheckM2
- **Snakemake** (Workflow engine executing the read-mapping evaluation steps within Mapler)

## Examples

```
minimap2 -ax map-hifi assembly.fasta reads.fastq | samtools sort -o reads_on_contigs.bam && samtools index reads_on_contigs.bam
```

## Evaluation signals

- read_contig_mapping.txt exists and reports aligned read and aligned base ratios split by bin quality (near complete, high, medium, low, unbinned)
- read_contig_mapping_plot.pdf is generated as the visual counterpart of read_contig_mapping.txt
- All reported ratios fall within [0, 1]
- Higher aligned ratios indicate a more representative assembly; on Deadwood, metaMDBG should show 52.7% unaligned bases vs 76.7% for metaFlye and 66.1% for hifiasm-meta
- reads_on_contigs.bam and its .bai index are produced from the Minimap2 alignment step

## Limitations

- If the aligned read ratio is significantly higher than the aligned base ratio, most reads are only partially aligned to contigs, so read-based ratios alone overstate assembly representativeness
- Low abundance reads are typically more abundant in the unmapped portions, so unmapped fractions may reflect low-abundance diversity rather than assembly failure
- Multi-contig reads are counted only at the highest bin quality level they touch, which conditions the binning-aware breakdown on the quality of available bins

## Evidence

- [readme] Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
- [methods] Higher ratios indicate representativeness: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] Partial alignment diagnostic: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [methods] Unmapped low-abundance reads: "Typically, low abundance reads are more libely to be more abundant in the unmapped portions"
- [readme] read_contig_mapping.txt description: "read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
