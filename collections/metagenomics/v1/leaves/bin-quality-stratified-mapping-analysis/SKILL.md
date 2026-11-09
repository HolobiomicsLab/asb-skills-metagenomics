---
name: bin-quality-stratified-mapping-analysis
description: Use when evaluating a HiFi metagenome assembly by aligning raw reads
  to contigs with minimap2 and breaking down mapping ratios by CheckM2 bin quality
  to benchmark assemblies (e.g. Mock vs Gut vs Deadwood).
when_to_use_negative:
- The input is raw sequencing data with no assembly available and assembly itself
  is the goal rather than evaluation (use the full Mapler assembly pipeline instead).
- You need taxonomy-aware evaluation of unmapped reads (Kraken 2/GTDB-Tk/k-mer analyses)
  rather than quality-stratified mapping ratios.
- Reads are short-read or non-HiFi; the pipeline targets HiFi long-read metagenome
  assemblies.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Mapler
  role: Snakemake pipeline orchestrating read-contig mapping evaluation and bin-quality-stratified
    reporting
  repo: https://github.com/Nimauric/Mapler
- name: minimap2
  role: Aligns HiFi reads on assembly contigs to produce reads_on_contigs.bam used
    for ratio computation
- name: CheckM2
  role: Estimates bin completeness/contamination so mapping ratios can be split by
    bin quality class
  repo: https://github.com/chklovski/CheckM2
- name: metaMDBG
  role: Source of the assemblies being evaluated (e.g. Mock, Gut, Deadwood datasets)
  repo: https://github.com/GaetanBenoitDev/metaMDBG
- name: MetaBAT2
  role: Performs binning whose quality classes feed the stratified mapping report
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/
- name: Snakemake
  role: Execution engine for the pipeline rules (read_mapping_evaluation, read_contig_mapping)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:9f7184e03dd83f2d15fa063228016bcfc6ab60748f08eca51d18f8862ac866f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Bash
  - CheckM2
  - Conda
  - Mapler
  - MetaBAT2
  - Python
  - Snakemake
  - metaMDBG
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# bin-quality-stratified-mapping-analysis

## Summary

Computes aligned read and aligned base ratios of HiFi reads on assembly contigs and splits them by bin quality class (near complete, high quality, medium quality, low quality) using CheckM2-supported binning, to quantify how much of the sequenced sample an assembly captures.

## When to use

Use when evaluating a HiFi metagenome assembly by aligning raw reads to contigs with minimap2 and breaking down mapping ratios by CheckM2 bin quality to benchmark assemblies (e.g. Mock vs Gut vs Deadwood).

## When NOT to use

- The input is raw sequencing data with no assembly available and assembly itself is the goal rather than evaluation (use the full Mapler assembly pipeline instead).
- You need taxonomy-aware evaluation of unmapped reads (Kraken 2/GTDB-Tk/k-mer analyses) rather than quality-stratified mapping ratios.
- Reads are short-read or non-HiFi; the pipeline targets HiFi long-read metagenome assemblies.

## Inputs

- Raw HiFi read FASTQ files (declared via read_path in the configfile)
- Assembly contigs in FASTA (outputs/<sample_name>/<custom_assembly_process>/assembly.fasta)
- Mapler configfile based on config/config_template.yaml (read_mapping_evaluation: true, binning: true, checkm: true)

## Outputs

- reads_on_contigs.bam and reads_on_contigs.bam.bai (minimap2 alignment of reads on contigs)
- reads_on_contigs_mapping_evaluation/report.txt (overall aligned read and aligned base ratios)
- read_contig_mapping.txt (aligned read/base ratios split by bin quality class)
- read_contig_mapping_plot.pdf (visual version of read_contig_mapping.txt)

## How to apply

Retrieve raw HiFi read FASTQ files and the assembly (e.g. precomputed metaMDBG assembly from doi:10.57745/2SA8AB), placing the assembly at outputs/<sample_name>/<custom_assembly_process>/assembly.fasta and declaring read_path entries in the configfile. Copy config/config_template.yaml and enable read_mapping_evaluation: true and binning: true with checkm: true, running under Snakemake >=8.28 with Conda in the 'mapler' environment. Align reads on contigs with minimap2 to produce reads_on_contigs.bam plus its .bai index, then run the reads_on_contigs_mapping_evaluation rule which writes aligned read ratio (aligned read count / total read count) and aligned base ratio (aligned read bases / total read length) to reads_on_contigs_mapping_evaluation/report.txt. Finally run the CheckM2-supported read_contig_mapping step to split the same ratios by bin quality class into read_contig_mapping.txt and read_contig_mapping_plot.pdf; a large gap between aligned read and aligned base ratios signals mostly partially aligned reads.

## Related tools

- **Mapler** (Snakemake pipeline orchestrating read-contig mapping evaluation and bin-quality-stratified reporting) — https://github.com/Nimauric/Mapler
- **minimap2** (Aligns HiFi reads on assembly contigs to produce reads_on_contigs.bam used for ratio computation)
- **CheckM2** (Estimates bin completeness/contamination so mapping ratios can be split by bin quality class) — https://github.com/chklovski/CheckM2
- **metaMDBG** (Source of the assemblies being evaluated (e.g. Mock, Gut, Deadwood datasets)) — https://github.com/GaetanBenoitDev/metaMDBG
- **MetaBAT2** (Performs binning whose quality classes feed the stratified mapping report) — https://bitbucket.org/berkeleylab/metabat/src/master/
- **Snakemake** (Execution engine for the pipeline rules (read_mapping_evaluation, read_contig_mapping))

## Examples

```
conda activate mapler && ./local_pipeline.sh config/config_template.yaml > mylog.txt
```

## Evaluation signals

- reads_on_contigs_mapping_evaluation/report.txt exists and contains aligned read and aligned base ratios.
- Published reference values are reproduced within tolerance: 96.9%/96.1% (Mock), 49.5%/47.8% (Gut), 12.6%/9.4% (Deadwood, medium quality or higher).
- Aligned read ratio should not be significantly higher than aligned base ratio; a large gap indicates most reads are only partially aligned to contigs.
- read_contig_mapping.txt contains ratios split across all four quality classes (near complete, high quality, medium quality, low quality).
- checkm_report.txt under the binning directory reports bin counts per quality category, confirming binning/checkm ran before stratification.

## Limitations

- Low abundance reads are more likely to be abundant in the unmapped portions, so low ratios can reflect community complexity rather than assembler failure.
- In the Deadwood dataset the metaMDBG assembly is of lower quality, so only 12.6% of reads and 9.4% of bases align with bins of medium quality or higher; stratified ratios degrade sharply for complex environmental samples.
- The custom assembly process name must not collide with built-in assemblers (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS) or the pipeline will not pick up the user assembly.

## Evidence

- [readme] read_contig_mapping.txt report: "read_contig_mapping.txt # A report on the aligned reads and aligned bases ratios, split by bin quality"
- [methods] ratios meaning: "The higher those ratios are, the more representative of the sequenced sample is the assembly."
- [methods] partial alignment signal: "If the aligned read ratio is significantly higher than the aligned base ratio, it is a sign that most reads are only partially aligned to contigs."
- [methods] unmapped bias: "Typically, low abundance reads are more libely to be more abundant in the unmapped portions"
- [readme] mapped/unmapped analysis: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
- [readme] install environment: "conda create -n mapler -c bioconda -c conda-forge 'bioconda::snakemake>=8.28' 'conda-forge::conda>=24.1.2' bioconda::snakemake-executor-plugin-slurm"
