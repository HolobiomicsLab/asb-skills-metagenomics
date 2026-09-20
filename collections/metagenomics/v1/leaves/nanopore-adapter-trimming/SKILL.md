---
name: nanopore-adapter-trimming
description: Use when raw Nanopore FASTQ reads enter a metagenome assembly workflow
  and adapters must be removed before control-read decontamination (NanoLyse), quality
  filtering (Filtlong) and long-read assembly.
when_to_use_negative:
- Input reads are PacBio — Porechop adapter trimming is skipped for PacBio reads in
  nf-core/mag conventions
- Reads are Illumina short reads — use short-read trimming tools (e.g. fastp) instead
  of the Nanopore-specific Porechop step
- Reads are already adapter-trimmed and only require control decontamination or length/quality
  filtering
edam_operation: http://edamontology.org/operation_3238
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3941
tools:
- name: Porechop
  role: Finds and removes Oxford Nanopore adapters; trims end adapters and chops chimeric
    reads with internal adapters
  repo: https://github.com/rrwick/Porechop
- name: nf-core/mag
  role: Pipeline that hosts the long-read pre-processing subworkflow in which Porechop
    adapter trimming is embedded and gated to Nanopore reads
  repo: https://github.com/nf-core/mag
- name: Filtlong
  role: Downstream quality filter that subsets the adapter-trimmed long reads by length
    and identity
  repo: https://github.com/rrwick/Filtlong
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:9471b782ee4191752ceab6894473e59b19ee83e6d36f97e008902910fd13cfc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rrwick/Porechop
  tools:
  - Filtlong
  - Porechop
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_3238
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nanopore-adapter-trimming

## Summary

Adapter trimming of raw Oxford Nanopore long reads with Porechop, as implemented in the nf-core/mag long-read pre-processing module, so that chimeric and adapter-containing reads are cleaned before decontamination, quality filtering and metagenome assembly.

## When to use

Use when raw Nanopore FASTQ reads enter a metagenome assembly workflow and adapters must be removed before control-read decontamination (NanoLyse), quality filtering (Filtlong) and long-read assembly.

## When NOT to use

- Input reads are PacBio — Porechop adapter trimming is skipped for PacBio reads in nf-core/mag conventions
- Reads are Illumina short reads — use short-read trimming tools (e.g. fastp) instead of the Nanopore-specific Porechop step
- Reads are already adapter-trimmed and only require control decontamination or length/quality filtering

## Inputs

- Raw Oxford Nanopore long-read FASTQ files
- Nanopore reads gated in the pipeline so the step is skipped for PacBio reads

## Outputs

- Adapter-trimmed Nanopore FASTQ (input to NanoLyse/Filtlong)
- QC summary statistics and reports for the raw and filtered read sets

## How to apply

Run Porechop on the raw Nanopore FASTQ as the first step of the nf-core/mag long-read pre-processing subworkflow: Porechop performs thorough alignments to find adapter sequences, even at low sequence identity, trims adapters from read ends, and chops reads carrying internal adapters (treated as chimeric) into separate reads. Gate this step to Nanopore data — in nf-core/mag, Porechop adapter trimming is skipped for PacBio reads. Feed the adapter-trimmed FASTQ into NanoLyse for phage-lambda control removal, then Filtlong for quality filtering, and generate QC reports (e.g. NanoPlot) for raw and filtered read sets. Validate by running the pipeline on nf-core test data so GitHub Actions tests exit successfully and the quality-filtered FASTQ is present as the stage output.

## Related tools

- **Porechop** (Finds and removes Oxford Nanopore adapters; trims end adapters and chops chimeric reads with internal adapters) — https://github.com/rrwick/Porechop
- **nf-core/mag** (Pipeline that hosts the long-read pre-processing subworkflow in which Porechop adapter trimming is embedded and gated to Nanopore reads) — https://github.com/nf-core/mag
- **Filtlong** (Downstream quality filter that subsets the adapter-trimmed long reads by length and identity) — https://github.com/rrwick/Filtlong

## Examples

```
porechop -i raw_nanopore_reads.fastq -o adapter_trimmed_reads.fastq
```

## Evaluation signals

- GitHub Actions pipeline tests run nf-core/mag on the test data and exit successfully
- The quality-filtered FASTQ is present as the stage output after Porechop → NanoLyse → Filtlong
- QC reports are generated for both the raw and the filtered read sets (NanoPlot-style summaries)
- Step is skipped (no Porechop output channel emitted) when the input is PacBio rather than Nanopore
- Trimmed reads no longer contain terminal adapter alignments detectable by Porechop

## Limitations

- Porechop is specific to Oxford Nanopore reads and is skipped for PacBio data in nf-core/mag
- Adapter trimming alone does not remove sequencing control contamination (phage lambda) — NanoLyse must run afterwards
- Internal adapters cause reads to be chopped as chimeric, which changes read counts and length distributions

## Evidence

- [readme] Porechop is a tool for finding and removing adapters from Oxford Nanopore reads.: [evidence span withheld — anchor status: not_found]
- [methods] Porechop adapter trimming is skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [intro] A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [readme] Adapters on the ends of reads are trimmed off, and when a read has an adapter in its middle, it is treated as chimeric: [evidence span withheld — anchor status: external_doc]
