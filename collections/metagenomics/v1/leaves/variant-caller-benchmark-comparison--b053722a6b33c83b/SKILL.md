---
name: variant-caller-benchmark-comparison
description: Decision support for choosing a variant caller for low-frequency (<0.01)
  SARS-CoV-2 mutation detection in wastewater FASTQ data when lineages must be inferred
  per variant caller before committing to one VCF source.
when_to_use_negative:
- The input is clinical isolate data with high-frequency variants only — freebayes
  at 0.01 mutation frequency is motivated by detecting variants below 1% frequency
  in mixed wastewater samples.
- A single caller's output has already been fixed for the project and no benchmarking
  decision is needed — this skill would duplicate existing variant calling.
- The sample fails preprocessing quality checks (e.g., insufficient paired-end read
  depth after trimming to a 200 nt minimum with sliding-window quality 15), making
  per-caller Nt counts incomparable.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: freebayes
  role: Variant caller run with low mutation frequency 0.01 to capture variants below
    1%; judged the most productive and informative and used for all subsequent results
- name: GATK Mutect2
  role: Variant caller run in cancer-only mode with default parameters as a benchmark
    arm
- name: mpileup
  role: Variant caller benchmark arm configured to report every position
- name: Minimap2
  role: Maps cleaned paired-end reads to NC_045512 with minimal chaining score 40,
    producing the shared BAM used by all three callers
- name: lineagespot
  role: R pipeline run on each caller's VCF to assign SARS-CoV-2 lineages (Pangolin
    rules / outbreak.info) and count Nt per lineage
  repo: github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:45b21eaeced75e5428fcd511b744305ac7ea682742f8344242ca34aa14acddb3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GATK Mutect2
  - Minimap2
  - R
  - freebayes
  - lineagespot
  - mpileup
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# variant-caller-benchmark-comparison

## Summary

Benchmark three SARS-CoV-2 variant callers (freebayes at low mutation frequency 0.01, GATK Mutect2 in cancer-only mode, and mpileup reporting every position) on the same wastewater alignment to compare lineage-characteristic mutation counts (Nt) and select the most productive caller for downstream lineagespot analysis.

## When to use

Decision support for choosing a variant caller for low-frequency (<0.01) SARS-CoV-2 mutation detection in wastewater FASTQ data when lineages must be inferred per variant caller before committing to one VCF source.

## When NOT to use

- The input is clinical isolate data with high-frequency variants only — freebayes at 0.01 mutation frequency is motivated by detecting variants below 1% frequency in mixed wastewater samples.
- A single caller's output has already been fixed for the project and no benchmarking decision is needed — this skill would duplicate existing variant calling.
- The sample fails preprocessing quality checks (e.g., insufficient paired-end read depth after trimming to a 200 nt minimum with sliding-window quality 15), making per-caller Nt counts incomparable.

## Inputs

- FASTQ reads of a wastewater sample (paired-end, Illumina ARTIC protocol) from ENA study PRJEB44141 (ERP128154)
- SARS-CoV-2 Wuhan reference genome NC_045512 (FASTA)
- Cleaned BAM alignment (reads mapped with Minimap2, chaining score 40; unpaired/multiple mappings removed)
- lineagespot lineage definitions (Pangolin rules / outbreak.info mutation profiles)

## Outputs

- Three per-caller VCF files (freebayes, GATK Mutect2, mpileup) from the same input BAM
- Per-caller Nt counts of unique nucleotide mutations per SARS-CoV-2 lineage (B.1.177, B.1.1.7, B.1.351)
- Side-by-side comparison table (variant_caller_comparison.csv) supporting the caller-selection decision

## How to apply

Download the FASTQ reads of the target wastewater time period (here, the 05–11 February 2021 sample from ENA study PRJEB44141/ERP128154) and map cleaned paired-end reads to the SARS-CoV-2 Wuhan reference genome NC_045512 with Minimap2 using a minimal chaining score (matching bases minus log gap penalty) of 40, following a variation of the Illumina ARTIC pipeline. Call variants on this single BAM with three callers: freebayes using a low mutation frequency parameter of 0.01, GATK Mutect2 in cancer-only mode with default parameters, and mpileup configured to report every position. Run the lineagespot R pipeline on each caller's VCF to identify lineages via Pangolin rules / outbreak.info definitions and count unique nucleotide mutations (Nt) per lineage. Tabulate Nt counts for lineages B.1.177, B.1.1.7, and B.1.351 side-by-side per caller and compare the differences; the caller yielding the richest, most informative set (freebayes in this study) becomes the VCF source for all subsequent results.

## Related tools

- **freebayes** (Variant caller run with low mutation frequency 0.01 to capture variants below 1%; judged the most productive and informative and used for all subsequent results)
- **GATK Mutect2** (Variant caller run in cancer-only mode with default parameters as a benchmark arm)
- **mpileup** (Variant caller benchmark arm configured to report every position)
- **Minimap2** (Maps cleaned paired-end reads to NC_045512 with minimal chaining score 40, producing the shared BAM used by all three callers)
- **lineagespot** (R pipeline run on each caller's VCF to assign SARS-CoV-2 lineages (Pangolin rules / outbreak.info) and count Nt per lineage) — github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
freebayes -f NC_045512.fa -F 0.01 sample_0211.bam > freebayes_0.01.vcf; gatk Mutect2 -R NC_045512.fa -I sample_0211.bam --tumor-sample-ID --cancer-mode -O mutect2.vcf
```

## Evaluation signals

- Each caller was run on the identical Minimap2-aligned BAM to NC_045512 (chaining score 40), so any Nt differences are attributable to the caller, not the alignment.
- Per-caller Nt counts for B.1.177, B.1.1.7 and B.1.351 reproduce the paper's comparison table and pairwise Nt differences.
- freebayes (freq 0.01) yields the highest / most informative Nt counts, supporting the conclusion that it is the most productive and informative tool for <0.01 frequency variants.
- Each VCF is valid input to lineagespot and produces lineage assignments consistent with Pangolin rules / outbreak.info definitions.
- Resulting comparison table (variant_caller_comparison.csv) contains one Nt value per lineage per caller with no missing lineage rows.

## Limitations

- Software versions for Minimap2, freebayes, mpileup, and GATK Mutect2 were not reported in the paper, complicating exact reproduction.
- Pangolin-based lineage assignment can produce incorrect assignments when different reads satisfy different rules of the same lineage.
- Mutation identification may be hindered by viral adsorption and decay in wastewater; preprocessing was inconsistent across samples (primer trimming and remapping were omitted for the four samples between 19 March and 15 April 2021).

## Evidence

- [results] three different variant callers were assessed: (1) freebayes14, (2) mpileup15 and (3) GATK Mutect216 (cancer only mode): "three different variant callers were assessed: (1) freebayes14, (2) mpileup15 and (3) GATK Mutect216 (cancer only mode)"
- [results] freebayes most productive finding: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] Minimap2 mapping parameters: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [readme] lineagespot input description: "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [intro] low mutation frequency parameter motivation: "Based on a variation of the Illumina ARTIC pipeline for the identification of mutations at low frequencies (< 0.01)"
