---
name: long-read-quality-filtering
description: Use when raw or decontaminated Oxford Nanopore FASTQ reads must be quality-filtered
  (Filtlong) and QC-reported (NanoPlot) before long-read or hybrid metagenome assembly
  in nf-core/mag.
when_to_use_negative:
- Input reads are PacBio, where nf-core/mag skips Porechop adapter trimming ('Porechop
  adapter trimming is skipped for PacBio reads'), so the Nanopore-gated pre-processing
  chain does not apply as-is
- Input is already a quality-filtered FASTQ (re-filtering risks losing reads needed
  for assembly depth)
- Short Illumina reads, which nf-core/mag pre-processes with a different short-read
  QC chain (e.g. fastp/FastQC) rather than the long-read module
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Filtlong
  role: Filters long reads by quality using both read length and read identity to
    produce a smaller, better subset FASTQ
  repo: https://github.com/rrwick/Filtlong
- name: NanoPlot
  role: Plots and summarizes QC metrics for long read sequencing data, applied to
    raw and filtered read sets
- name: Porechop
  role: Upstream adapter trimming of Nanopore reads that gates this filtering step
  repo: https://github.com/rrwick/Porechop
- name: nf-core/mag
  role: Pipeline whose long-read pre-processing subworkflow orchestrates the filtering
    and QC stages
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:cbb38548489c036d72489a67c45e6afde452616b1b81933c1ae025ad02eb2557
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rrwick/Filtlong
  tools:
  - Filtlong
  - NanoPlot
  - Porechop
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# long-read-quality-filtering

## Summary

Quality-filter decontaminated long Nanopore reads with Filtlong and generate QC reports for raw and filtered read sets as implemented in the nf-core/mag long-read pre-processing subworkflow, so that only the best length/identity reads feed downstream assembly.

## When to use

Use when raw or decontaminated Oxford Nanopore FASTQ reads must be quality-filtered (Filtlong) and QC-reported (NanoPlot) before long-read or hybrid metagenome assembly in nf-core/mag.

## When NOT to use

- Input reads are PacBio, where nf-core/mag skips Porechop adapter trimming ('Porechop adapter trimming is skipped for PacBio reads'), so the Nanopore-gated pre-processing chain does not apply as-is
- Input is already a quality-filtered FASTQ (re-filtering risks losing reads needed for assembly depth)
- Short Illumina reads, which nf-core/mag pre-processes with a different short-read QC chain (e.g. fastp/FastQC) rather than the long-read module

## Inputs

- Long-read FASTQ (Oxford Nanopore) reads, adapter-trimmed by Porechop and decontaminated with NanoLyse against the phage lambda control reference
- Raw Nanopore FASTQ read set for baseline QC reporting

## Outputs

- Quality-filtered Nanopore FASTQ (stage output feeding assembly)
- NanoPlot QC summary statistics and HTML reports for raw and filtered read sets

## How to apply

In the nf-core/mag pipeline, run this step after Porechop adapter trimming and NanoLyse removal of phage-lambda control reads, so Filtlong receives only decontaminated long reads. Filtlong filters the Nanopore FASTQ using both read length and read identity, producing a smaller, higher-quality subset as the stage output FASTQ. NanoPlot is run to generate QC summary statistics and reports for both the raw input read set and the quality-filtered read set, giving before/after comparison. Porechop adapter trimming is gated to Nanopore reads and skipped for PacBio reads per nf-core/mag conventions, so apply this filtering chain conditionally on the input technology. Validate by running the pipeline's test data through GitHub Actions and confirming the quality-filtered FASTQ is present as the stage output.

## Related tools

- **Filtlong** (Filters long reads by quality using both read length and read identity to produce a smaller, better subset FASTQ) — https://github.com/rrwick/Filtlong
- **NanoPlot** (Plots and summarizes QC metrics for long read sequencing data, applied to raw and filtered read sets)
- **Porechop** (Upstream adapter trimming of Nanopore reads that gates this filtering step) — https://github.com/rrwick/Porechop
- **nf-core/mag** (Pipeline whose long-read pre-processing subworkflow orchestrates the filtering and QC stages) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag -profile test,docker --input samplesheet.csv
```

## Evaluation signals

- Pipeline test run (GitHub Actions) on the bundled test data exits successfully
- The quality-filtered FASTQ is present as the stage output of the pre-processing subworkflow
- NanoPlot QC reports exist for both the raw and the filtered read sets, allowing before/after comparison of read length and quality distributions
- Filtered read set is a subset of the decontaminated input (no reads absent from the NanoLyse-cleaned FASTQ)
- Read count and total bases of the filtered FASTQ are lower than or equal to the decontaminated input, consistent with Filtlong's 'smaller, better subset' behavior

## Limitations

- Filtlong discards reads, reducing total bases; over-aggressive filtering can starve downstream long-read or hybrid assembly
- The Nanopore-specific gating means the module cannot be applied unchanged to PacBio input
- Filtering is quality-only; removal of phage lambda control contamination must be done upstream with NanoLyse, not by Filtlong

## Evidence

- [readme] Filtlong filters long reads by quality, using length and identity: [evidence span withheld — anchor status: external_doc]
- [readme] Filtlong combines length and identity criteria: [evidence span withheld — anchor status: external_doc]
- [intro] Metagenomic workflow begins with raw-read preprocessing: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [methods] Nanopore-specific gating of adapter trimming in nf-core/mag: "Porechop adapter trimming is skipped for PacBio reads"
- [readme] NanoPlot plots long-read sequencing QC: [evidence span withheld — anchor status: external_doc]
