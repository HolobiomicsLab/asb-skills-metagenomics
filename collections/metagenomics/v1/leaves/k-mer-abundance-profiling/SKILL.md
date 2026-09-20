---
name: k-mer-abundance-profiling
description: 'Use when diagnosing why HiFi reads failed to map to a metagenome assembly:
  compare k-mer abundance histograms of mapped vs unmapped read fractions to test
  the low-abundance hypothesis.'
when_to_use_negative:
- The question is taxonomic composition of unassembled reads (use the Kraken 2/Krona
  fraction of the pipeline, not KAT)
- The goal is bin completeness/contamination assessment (use CheckM/GTDB-Tk bin classification
  instead)
- Reads are short-read data; the Mapler/KAT workflow here is designed for HiFi long-read
  metagenome assemblies
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Mapler
  role: Snakemake pipeline that splits HiFi reads into mapped/unmapped fractions via
    minimap2 and orchestrates the per-fraction k-mer frequency analysis
  repo: https://github.com/Nimauric/Mapler
- name: KAT
  role: K-mer Analysis Toolkit computing k-mer frequency histograms and median k-mer
    abundance per read fraction (kat-plot.pdf, <fraction_name>-stats.tsv)
  repo: https://github.com/EarlhamInst/KAT
- name: minimap2
  role: Aligns HiFi reads on assembly contigs to define the mapped and unmapped read
    fractions prior to k-mer profiling
- name: FastQC
  role: Provides per-fraction read length and quality summaries used to rule out length/quality
    as causes of non-assembly
  repo: https://github.com/s-andrews/FastQC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:4b20e0a49f82a32ac15dce60abe1345ec0f4eca48324176402e6c037f3de8429
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
  - minimap2
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# k-mer-abundance-profiling

## Summary

Profile k-mer frequency distributions of mapped and unmapped read fractions from HiFi metagenome assemblies using KAT, to determine whether unassembled reads are low-abundance (e.g., dominated by abundance-1 k-mers) rather than degraded in length or quality.

## When to use

Use when diagnosing why HiFi reads failed to map to a metagenome assembly: compare k-mer abundance histograms of mapped vs unmapped read fractions to test the low-abundance hypothesis.

## When NOT to use

- The question is taxonomic composition of unassembled reads (use the Kraken 2/Krona fraction of the pipeline, not KAT)
- The goal is bin completeness/contamination assessment (use CheckM/GTDB-Tk bin classification instead)
- Reads are short-read data; the Mapler/KAT workflow here is designed for HiFi long-read metagenome assemblies

## Inputs

- Mapped read fraction FASTQ (e.g., mapped_reads.fastq) produced by Mapler read-contig alignment
- Unmapped read fraction FASTQ (e.g., unmapped_reads.fastq) produced by Mapler read-contig alignment
- Full HiFi read set (e.g., Deadwood PacBio HiFi reads from doi:10.57745/2SA8AB) as k-mer counting reference
- Mapler configfile (YAML) with kat: true and both mapped and unmapped fractions enabled

## Outputs

- kat-plot.pdf comparing k-mer abundance distributions of mapped vs unmapped fractions
- <fraction_name>-stats.tsv per-fraction k-mer statistics (median k-mer abundance)
- Fraction-comparison summary table entry recording median k-mer abundance per fraction

## How to apply

Within the Mapler Snakemake pipeline, first align Deadwood HiFi reads onto the metaMDBG assembly (assembly.fasta) with minimap2 and emit the mapped and unmapped read fractions as <fraction_name>_reads.fastq under outputs/<sample_name>/<assembler>/. Enable KAT in the Mapler configfile (kat: true) so that k-mer frequency analysis runs separately on each read fraction; a kat-plot.pdf is produced when both 'mapped' and 'unmapped' fractions are requested. Interpret the KAT statistics (<fraction_name>-stats.tsv): an unmapped fraction whose k-mer distribution is dominated by abundance-1 k-mers (median k-mer abundance of 1) indicates low genomic abundance, not read quality or length, explains the assembly failure. Cross-check with FastQC average read lengths (e.g., 18,583 bp unmapped vs 18,437 bp mapped) to rule out length bias. Compile the per-fraction k-mer abundance metrics into a fraction-comparison summary table alongside read_contig_mapping.txt and report.txt.

## Related tools

- **Mapler** (Snakemake pipeline that splits HiFi reads into mapped/unmapped fractions via minimap2 and orchestrates the per-fraction k-mer frequency analysis) — https://github.com/Nimauric/Mapler
- **KAT** (K-mer Analysis Toolkit computing k-mer frequency histograms and median k-mer abundance per read fraction (kat-plot.pdf, <fraction_name>-stats.tsv)) — https://github.com/EarlhamInst/KAT
- **minimap2** (Aligns HiFi reads on assembly contigs to define the mapped and unmapped read fractions prior to k-mer profiling)
- **FastQC** (Provides per-fraction read length and quality summaries used to rule out length/quality as causes of non-assembly) — https://github.com/s-andrews/FastQC

## Examples

```
./local_pipeline.sh config/config_deadwood.yaml > mylog.txt  # configfile sets fractions: [mapped, unmapped], fastqc: true, kat: true, kraken2: true
```

## Evaluation signals

- Unmapped fraction KAT distribution is dominated by abundance-1 k-mers (median k-mer abundance of 1)
- FastQC average read length of unmapped reads is comparable to mapped reads (e.g., 18,583 vs 18,437 bp in Deadwood), ruling out length bias
- kat-plot.pdf is generated only when both 'mapped' and 'unmapped' fractions are configured
- Per-fraction stats (<fraction_name>-stats.tsv) reproduce reported values within expected stochastic/deviation tolerance
- read_contig_mapping.txt aligned read/base ratios are consistent with the fraction split used for k-mer profiling

## Limitations

- KAT kat-plot.pdf requires both mapped and unmapped fractions; running a single fraction yields no comparative plot
- Median abundance of 1 indicates low abundance but does not by itself exclude other causes without the FastQC length/quality cross-check
- Only part of the analysis runs on the bundled test dataset; full-fraction profiling requires the user's own configfile and external read data

## Evidence

- [methods] KAT k-mer abundance comparison: "separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment"
- [methods] Low-abundance rationale: "Typically, low abundance reads are more libely to be more abundant in the unmapped portions"
- [readme] KAT output conditions: "├── kat # Requires kat: true
│   │   ├── kat-plot.pdf # Requires both "mapped" and "unmapped" fractions"
- [readme] KAT tool scope: [evidence span withheld — anchor status: external_doc]
- [readme] Pipeline alignment step: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases"
