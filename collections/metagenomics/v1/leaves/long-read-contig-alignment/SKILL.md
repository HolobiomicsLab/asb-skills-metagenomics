---
name: long-read-contig-alignment
description: Use when evaluating how much of a HiFi read set a metagenome assembly
  (e.g. metaMDBG, metaflye, hifiasm-meta, or a user-submitted assembly.fasta) actually
  captures, via read-to-contig alignment ratios.
when_to_use_negative:
- You only need assembly contiguity statistics (N50, length) rather than how much
  of the sequenced sample the assembly represents — MetaQUAST is the appropriate Mapler
  module.
- Reads are short reads, not HiFi long reads — the pipeline and metrics are built
  around HiFi long-read metagenomic assemblies.
- The assembly is not available and only reads are provided without enabling an assembler
  — Mapler needs either a built-in assembler run or a user-submitted assembly.fasta
  to align against.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Mapler
  role: Snakemake pipeline that orchestrates the alignment, ratio computation and
    bin-quality-aware reporting
  repo: https://github.com/Nimauric/Mapler
- name: minimap2
  role: Aligns the HiFi long reads to the assembly contigs into reads_on_contigs.bam
- name: CheckM2
  role: Supplies quality_report.tsv used to classify bins (near complete / high /
    medium / low quality) for the binning-aware ratio split
  repo: https://github.com/chklovski/CheckM2
- name: Snakemake
  role: Execution engine for the Mapler workflow (conda environment 'mapler' with
    bioconda::snakemake>=8.28)
- name: metaMDBG
  role: Produces the assembly.fasta contigs that reads are aligned against
  repo: https://github.com/GaetanBenoitDev/metaMDBG
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:0fd2aacff195842f52cfd3547c22a10d69c76339494e304d1586c0558aa5dabb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Bash
  - CheckM2
  - Conda
  - FastQC
  - KAT
  - Kraken2
  - Mapler
  - Python
  - Snakemake
  - metaMDBG
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# long-read-contig-alignment

## Summary

Aligns HiFi long reads to metagenome assembly contigs with minimap2 (producing reads_on_contigs.bam + .bai) and computes aligned read and aligned base ratios, optionally split by bin quality category, as core assembly-representativeness metrics of the Mapler pipeline.

## When to use

Use when evaluating how much of a HiFi read set a metagenome assembly (e.g. metaMDBG, metaflye, hifiasm-meta, or a user-submitted assembly.fasta) actually captures, via read-to-contig alignment ratios.

## When NOT to use

- You only need assembly contiguity statistics (N50, length) rather than how much of the sequenced sample the assembly represents — MetaQUAST is the appropriate Mapler module.
- Reads are short reads, not HiFi long reads — the pipeline and metrics are built around HiFi long-read metagenomic assemblies.
- The assembly is not available and only reads are provided without enabling an assembler — Mapler needs either a built-in assembler run or a user-submitted assembly.fasta to align against.

## Inputs

- HiFi long-read FASTQ files (read_path entries in the Mapler configfile)
- Assembly contigs FASTA at outputs/<sample_name>/<assembler>/assembly.fasta
- Mapler configfile (copy of config/config_template.yaml) with read_mapping_evaluation: true
- CheckM2 quality_report.tsv (for the binning-aware split of ratios)

## Outputs

- reads_on_contigs.bam and reads_on_contigs.bam.bai (minimap2 alignment)
- reads_on_contigs_mapping_evaluation/report.txt (global aligned read and base ratios)
- read_contig_mapping.txt (ratios split by bin quality category)
- read_contig_mapping_plot.pdf (visual counterpart of the text report)

## How to apply

Place the assembly at outputs/<sample_name>/<assembler>/assembly.fasta, declare the HiFi reads via read_path in a Mapler configfile, and enable read_mapping_evaluation: true (plus binning: true with checkm: true for the binning-aware variant). Minimap2 aligns reads to contigs into reads_on_contigs.bam with a .bai index. The reads_on_contigs_mapping_evaluation rule computes the aligned read ratio (aligned read count / total read count) and the aligned base ratio (aligned read bases / total read length), writing report.txt. In the binning-aware version, contigs are assigned to bins classified by CheckM2 quality_report.tsv into near complete, high, medium or low quality (unbinned contigs form their own category); multi-contig reads are counted only at the highest bin quality level touched, and per-category ratios are aggregated into read_contig_mapping.txt (pandas) and its matplotlib counterpart read_contig_mapping_plot.pdf. Higher ratios mean a more representative assembly; a read ratio significantly above the base ratio signals mostly partially aligned reads.

## Related tools

- **Mapler** (Snakemake pipeline that orchestrates the alignment, ratio computation and bin-quality-aware reporting) — https://github.com/Nimauric/Mapler
- **minimap2** (Aligns the HiFi long reads to the assembly contigs into reads_on_contigs.bam)
- **CheckM2** (Supplies quality_report.tsv used to classify bins (near complete / high / medium / low quality) for the binning-aware ratio split) — https://github.com/chklovski/CheckM2
- **Snakemake** (Execution engine for the Mapler workflow (conda environment 'mapler' with bioconda::snakemake>=8.28))
- **metaMDBG** (Produces the assembly.fasta contigs that reads are aligned against) — https://github.com/GaetanBenoitDev/metaMDBG

## Examples

```
./local_pipeline.sh config/config_template.yaml > mylog.txt
```

## Evaluation signals

- report.txt exists under outputs/<sample_name>/<assembler>/reads_on_contigs_mapping_evaluation/ and both ratios fall within [0, 1].
- Aligned base ratio is not drastically lower than the aligned read ratio; a large gap indicates most reads are only partially aligned to contigs.
- read_contig_mapping.txt reports ratios split by bin quality class (near complete, high, medium, low, unbinned) and read_contig_mapping_plot.pdf is generated as its visual counterpart.
- Reproduced benchmark values match published references within tolerance: 96.9%/96.1% (Mock), 49.5%/47.8% (Gut), 12.6%/9.4% at medium quality or higher (Deadwood).
- Higher aligned ratios consistently correspond to a more representative assembly of the sequenced sample.

## Limitations

- Aligned read and base ratios depend on binning quality when using the binning-aware variant; reads spanning contigs from different bins are credited only to the highest quality bin touched.
- In complex environmental samples (e.g. Deadwood with hundreds to thousands of populations), assemblies are of lower quality and only a small fraction of reads may align (12.6% of reads, 9.4% of bases at medium quality or higher).
- The pipeline requires Snakemake (>=8.28) and Conda, plus the SLURM executor plugin on clusters; working directory may need 'snakemake --unlock' after a crash.

## Evidence

- [methods] Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
- [methods] The higher those ratios are, the more representative of the sequenced sample is the assembly.: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs.: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [readme] read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality: "read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
- [methods] The pipeline requires Snakemake and Conda.: "The pipeline requires Snakemake and Conda."
