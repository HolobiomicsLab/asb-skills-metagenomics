---
name: assembly-benchmark-comparison-against-reference
description: Use when validating that a targeted assembly tool recovers comparable
  gene-containing scaffolds and gene diversity to full de novo assembly on the same
  samples, at lower computational cost.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler/ARGextender is designed to only
  work with short-read sequencing data
- The goal is whole-community profiling (taxonomy/abundance) rather than evaluating
  assembly recovery of targeted genes
- No reference gene database exists for the targets of interest, since both arms of
  the benchmark are matched against the same database
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_0769
tools:
- name: ARGextender
  role: Targeted de novo assembly arm of the benchmark; combines recursive KMA and
    SPAdes to recruit reads around ARGs
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: SPAdes
  role: Full metagenomic de novo assembly arm of the benchmark, used as the reference-standard
    comparison
- name: KMA
  role: Used both inside ARGextender for read recruitment and to match ResFinder database
    sequences to scaffold sets (with '-ont' parameter)
- name: ARGprofiler
  role: Snakemake workflow wrapping the tools that generate the assemblies being benchmarked
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:cb2cf2059aa59f7ddf6ce7016fbfaf3a160b3bfefac68e96ff953bc3dd5d9d03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - SPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# assembly-benchmark-comparison-against-reference

## Summary

Benchmark a targeted de novo assembly method (ARGextender, combining KMA and SPAdes) against a full metagenomic de novo assembly (SPAdes) by matching a reference gene database (ResFinder/PanRes) to both outputs and comparing per-sample scaffold counts and distinct gene detections across a large cohort (951 urban sewage samples).

## When to use

Use when validating that a targeted assembly tool recovers comparable gene-containing scaffolds and gene diversity to full de novo assembly on the same samples, at lower computational cost.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler/ARGextender is designed to only work with short-read sequencing data
- The goal is whole-community profiling (taxonomy/abundance) rather than evaluating assembly recovery of targeted genes
- No reference gene database exists for the targets of interest, since both arms of the benchmark are matched against the same database

## Inputs

- Raw short-read sequencing data (FASTQ) for each sample (e.g. 951 urban sewage samples from ENA projects PRJEB40798, PRJEB40816, PRJEB40815, PRJEB27621, ERP015409)
- Published full de novo SPAdes assemblies of the same samples (FASTA scaffolds) for comparison
- Reference ARG database sequences (ResFinder / PanRes multi-FASTA)
- KMA 1.4.12a and SPAdes 3.15.5

## Outputs

- Per-sample counts of ARG-containing scaffolds for each tool (targeted vs full assembly)
- Per-sample counts of distinct ARGs detected by each tool
- Summary comparison table with per-tool means and ranges (e.g. ARGextender mean 101 scaffolds, range 1-350, vs SPAdes 95, range 1-348; 57 vs 58 distinct ARGs)
- Extracted flanking regions surrounding matched ARGs from both assembly types

## How to apply

Run the targeted assembly tool (ARGextender with recursive KMA 1.4.12a and SPAdes 3.15.5, unlimited recursions) on each sample's short reads, and obtain or produce full de novo SPAdes assemblies of the same reads (published assemblies can be downloaded from ENA, e.g. projects PRJEB40798, PRJEB40816, PRJEB40815, PRJEB27621, ERP015409). Match the reference database sequences (ResFinder/PanRes) to both scaffold sets using KMA 1.4.12a with the '-ont' parameter, extract flanking regions around matched ARGs, and count per sample the number of gene-containing scaffolds and distinct genes detected. Compute per-tool means and ranges and tabulate side by side; rationale is that targeted assembly avoids running the expensive full assembly unless targets are present, so equivalence in recovery metrics with reduced runtime justifies the targeted approach.

## Related tools

- **ARGextender** (Targeted de novo assembly arm of the benchmark; combines recursive KMA and SPAdes to recruit reads around ARGs) — https://github.com/genomicepidemiology/ARGprofiler
- **SPAdes** (Full metagenomic de novo assembly arm of the benchmark, used as the reference-standard comparison)
- **KMA** (Used both inside ARGextender for read recruitment and to match ResFinder database sequences to scaffold sets (with '-ont' parameter))
- **ARGprofiler** (Snakemake workflow wrapping the tools that generate the assemblies being benchmarked) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
kma -ont -t_db ResFinder -i sample_1.fq.gz sample_2.fq.gz -o kma_match; spades.py --only-assembler -1 sample_1.fq.gz -2 sample_2.fq.gz -o spades_out
```

## Evaluation signals

- Summary table reproduces reported figures: mean ~101 ARG-scaffolds for ARGextender (range 1-350) vs 95 for SPAdes (range 1-348)
- Distinct-ARG counts match: mean 57 for ARGextender (range 1-141) vs 58 for SPAdes (range 1-153)
- ARGextender runs in a much shorter time frame than full SPAdes assembly while producing comparable flanking regions
- Match step completed identically for both arms (same ResFinder database, KMA 1.4.12a with '-ont' parameter)
- Per-sample counts are computed consistently for all 951 samples across both tools

## Limitations

- ARGextender/ARGprofiler only supports short-read sequencing data; long-read technologies are not utilized
- Comparison depends on the choice of reference database (ResFinder/PanRes); PanRes combines different sources and scopes, so subset appropriateness is left to the research question
- Avoiding the expensive assembly step is only beneficial when target genes are actually identified in a sample

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [discussion] ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [discussion] This approach also avoids running the more expensive algorithm if none of the target genes is identified: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies: "ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies"
