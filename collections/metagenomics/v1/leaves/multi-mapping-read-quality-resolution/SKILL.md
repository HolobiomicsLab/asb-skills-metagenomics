---
name: multi-mapping-read-quality-resolution
description: Use when evaluating a HiFi metagenome assembly with reads that map to
  contigs belonging to multiple bins of different CheckM2 quality levels, and you
  need non-double-counted aligned read/base ratios split by bin quality.
when_to_use_negative:
- Input is only an unfiltered global aligned/unaligned ratio (report.txt of reads_on_contigs_mapping_evaluation)
  with no binning information — the simple, non-binning-aware metric suffices and
  no multi-mapping resolution is needed.
- No bin quality classification (e.g., no CheckM2 quality_report.tsv) is available,
  so contigs cannot be assigned to quality categories.
- Assembly is of an isolate or low-diversity sample where per-bin-quality breakdown
  of mapping ratios adds no information over global ratios.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Mapler
  role: Pipeline implementing the binning-aware read-to-contig mapping evaluation
    and its bin-quality resolution logic
  repo: https://github.com/Nimauric/Mapler
- name: minimap2
  role: Aligns HiFi long reads to assembly contigs to produce reads_on_contigs.bam
    used as input
- name: CheckM2
  role: Provides quality_report.tsv whose completeness/contamination values define
    the near complete / high / medium / low quality bin categories
  repo: https://github.com/chklovski/CheckM2
- name: Snakemake
  role: Workflow engine wiring the alignment, quality classification, and reporting
    steps together
- name: Python
  role: Implementation language using pysam for BAM parsing, pandas for report aggregation,
    and matplotlib for the PDF plot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:ac1d04a4efdc8200f4e95bda909d79c840627d0c5a33fca716346fdc88d7b76d
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
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# multi-mapping-read-quality-resolution

## Summary

Binning-aware metric computation from Mapler that resolves reads mapping to multiple contigs into a single bin-quality category (near complete > high > medium > low > unbinned), then reports aligned read and aligned base ratios per category so assembly representativeness can be judged per genome quality tier.

## When to use

Use when evaluating a HiFi metagenome assembly with reads that map to contigs belonging to multiple bins of different CheckM2 quality levels, and you need non-double-counted aligned read/base ratios split by bin quality.

## When NOT to use

- Input is only an unfiltered global aligned/unaligned ratio (report.txt of reads_on_contigs_mapping_evaluation) with no binning information — the simple, non-binning-aware metric suffices and no multi-mapping resolution is needed.
- No bin quality classification (e.g., no CheckM2 quality_report.tsv) is available, so contigs cannot be assigned to quality categories.
- Assembly is of an isolate or low-diversity sample where per-bin-quality breakdown of mapping ratios adds no information over global ratios.

## Inputs

- Sorted BAM of HiFi reads aligned to assembly contigs (reads_on_contigs.bam with .bai) from minimap2
- CheckM2 quality_report.tsv with completeness and contamination per bin
- Bin FASTA files mapping contig names to bins
- Read lengths / total read length of the input FASTQ

## Outputs

- read_contig_mapping.txt — text report of aligned read and aligned base ratios split by bin quality (near complete, high, medium, low, unbinned)
- read_contig_mapping_plot.pdf — matplotlib visual counterpart of read_contig_mapping.txt

## How to apply

Align the HiFi long reads to the assembly contigs with minimap2, producing a sorted BAM (reads_on_contigs.bam with .bai) handled via pysam. Parse the CheckM2 quality_report.tsv to classify each bin as near complete, high quality, medium quality, or low quality, as also reported in checkm_report.txt. Assign every contig to its bin's quality category, treating unbinned contigs as their own category; for reads with multiple mapped contigs, resolve each read to the highest bin quality level it touches so it is counted only once. Per category, compute the aligned read ratio (aligned read count / total read count) and the aligned base ratio (aligned read bases / total read length), then aggregate the per-category ratios into read_contig_mapping.txt with pandas and render the same breakdown graphically with matplotlib as read_contig_mapping_plot.pdf. The rationale is that a significantly higher aligned read ratio than aligned base ratio signals mostly partially aligned reads, and low abundance reads tend to dominate the unmapped portion.

## Related tools

- **Mapler** (Pipeline implementing the binning-aware read-to-contig mapping evaluation and its bin-quality resolution logic) — https://github.com/Nimauric/Mapler
- **minimap2** (Aligns HiFi long reads to assembly contigs to produce reads_on_contigs.bam used as input)
- **CheckM2** (Provides quality_report.tsv whose completeness/contamination values define the near complete / high / medium / low quality bin categories) — https://github.com/chklovski/CheckM2
- **Snakemake** (Workflow engine wiring the alignment, quality classification, and reporting steps together)
- **Python** (Implementation language using pysam for BAM parsing, pandas for report aggregation, and matplotlib for the PDF plot)

## Examples

```
minimap2 -ax map-hifi assembly.fasta reads.fastq | samtools sort -o reads_on_contigs.bam && samtools index reads_on_contigs.bam
```

## Evaluation signals

- read_contig_mapping.txt exists and reports aligned read and aligned base ratios explicitly split by bin quality category (near complete, high, medium, low, unbinned)
- read_contig_mapping_plot.pdf is generated as the visual counterpart of read_contig_mapping.txt
- All reported ratios fall within [0, 1]
- Higher aligned ratios indicate a more representative assembly; a ratio sum consistent with each multi-contig read counted only once (at its highest touched bin quality level)
- checkm_report.txt counts of near complete, high quality, medium quality and low quality bins match the categories used in the mapping report

## Limitations

- If the aligned read ratio is significantly higher than the aligned base ratio, most reads are only partially aligned to contigs, so the read-count ratio alone overstates assembly representativeness.
- Low abundance reads are more likely to be more abundant in the unmapped portions, so categories dominated by low-abundance bins may show low aligned ratios.
- The binning-aware breakdown depends on MetaBAT2 binning and CheckM2 quality estimates; errors in binning propagate into the category assignment of contigs.

## Evidence

- [methods] In the binning-aware version, the aligned percentage is separately calculated for reads or bases aligning to contigs belonging to bins of near complete, high, medium or low quality, or to unbinned contigs, with multi-contig reads counted only at the highest bin quality level.: [evidence span withheld — anchor status: not_found]
- [methods] Higher aligned ratios indicate a more representative assembly: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] Partially aligned reads interpretation: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [readme] Output report and plot description: "read_contig_mapping_plot.pdf # A visual version of read_contig_mapping.txt | read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
- [readme] Mapler analyzes the alignment of reads on contigs by calculating mapped read and base ratios: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
