---
name: phage-lambda-control-read-removal
description: Use when raw Nanopore metagenomic FASTQ reads contain phage-lambda sequencing
  control contamination and must be decontaminated after adapter trimming and before
  quality filtering and assembly.
when_to_use_negative:
- Input reads are PacBio, not Nanopore — Porechop adapter trimming is explicitly skipped
  for PacBio reads in nf-core/mag, and the Nanopore control-removal gating does not
  apply.
- Input is already adapter-trimmed and control-free (e.g. an already quality-filtered
  FASTQ) — re-running removal would not change the read set.
- The goal is short-read (Illumina) preprocessing — nf-core/mag handles short reads
  with a separate short-read QC path rather than the Porechop/NanoLyse/Filtlong long-read
  chain.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Porechop
  role: Trims adapters from Nanopore reads before control-read screening; skipped
    for PacBio reads
  repo: https://github.com/rrwick/Porechop
- name: NanoLyse
  role: Removes phage-lambda control contamination from the trimmed long reads
- name: Filtlong
  role: Quality-filters the decontaminated reads downstream, choosing reads by length
    and identity
  repo: https://github.com/rrwick/Filtlong
- name: NanoPlot
  role: Generates QC summary statistics and reports for raw and filtered read sets
- name: nf-core/mag
  role: Hosts the Nextflow long-read pre-processing subworkflow that wires these steps
    together
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:a7e94a7f0f458bf221276f1dfff638741fa5fb3fd6a9de4815f07f78ba054d1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rrwick/Porechop
  tools:
  - Filtlong
  - NanoLyse
  - NanoPlot
  - Porechop
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# phage-lambda-control-read-removal

## Summary

Removal of phage-lambda sequencing-control contamination from raw Oxford Nanopore metagenomic reads within the nf-core/mag long-read pre-processing subworkflow, so that downstream Filtlong quality filtering, NanoPlot QC, and assembly operate only on true sample reads.

## When to use

Use when raw Nanopore metagenomic FASTQ reads contain phage-lambda sequencing control contamination and must be decontaminated after adapter trimming and before quality filtering and assembly.

## When NOT to use

- Input reads are PacBio, not Nanopore — Porechop adapter trimming is explicitly skipped for PacBio reads in nf-core/mag, and the Nanopore control-removal gating does not apply.
- Input is already adapter-trimmed and control-free (e.g. an already quality-filtered FASTQ) — re-running removal would not change the read set.
- The goal is short-read (Illumina) preprocessing — nf-core/mag handles short reads with a separate short-read QC path rather than the Porechop/NanoLyse/Filtlong long-read chain.

## Inputs

- raw Oxford Nanopore metagenomic reads (FASTQ)
- lambda control reference for screening contaminated reads
- nf-core/mag long-read pre-processing subworkflow (Nextflow/DSL2)

## Outputs

- adapter-trimmed, lambda-control-decontaminated long-read FASTQ
- quality-filtered long-read FASTQ (Filtlong output, stage output)
- QC summary statistics and NanoPlot reports for raw and filtered read sets

## How to apply

In the nf-core/mag long-read pre-processing subworkflow, run the Nanopore-specific steps in order: first trim adapters from the raw long reads with Porechop (this step is gated to Nanopore reads and skipped for PacBio reads per nf-core/mag conventions), then screen the trimmed reads against the lambda control reference to remove phage-lambda control reads (NanoLyse-based decontamination per the traced finding). After decontamination, apply Filtlong quality filtering, which selects a smaller, better subset of long reads using both read length and read identity, and generate NanoPlot QC summary statistics and reports for both the raw and filtered read sets. The stage output must include the quality-filtered FASTQ, which feeds the downstream assembly.

## Related tools

- **Porechop** (Trims adapters from Nanopore reads before control-read screening; skipped for PacBio reads) — https://github.com/rrwick/Porechop
- **NanoLyse** (Removes phage-lambda control contamination from the trimmed long reads)
- **Filtlong** (Quality-filters the decontaminated reads downstream, choosing reads by length and identity) — https://github.com/rrwick/Filtlong
- **NanoPlot** (Generates QC summary statistics and reports for raw and filtered read sets)
- **nf-core/mag** (Hosts the Nextflow long-read pre-processing subworkflow that wires these steps together) — https://github.com/nf-core/mag

## Evaluation signals

- GitHub Actions pipeline tests run the nf-core/mag pipeline on the test data and exit successfully.
- The quality-filtered FASTQ is present as the stage output of the long-read pre-processing subworkflow.
- No lambda control reads remain in the decontaminated FASTQ (screened reads against the lambda control reference).
- QC reports exist for both the raw and the filtered read sets, showing a measurable reduction in reads after control removal and Filtlong filtering.
- The pipeline exits with zero error status after the long-read pre-processing subworkflow completes.

## Limitations

- Control-read removal is gated to Nanopore reads; PacBio inputs skip the Porechop trimming step, so the same chain cannot be applied unchanged to PacBio data.
- Removal quality depends on screening against the lambda control reference — control variants absent from the reference will pass through.
- Removal must occur before Filtlong, since Filtlong's length/identity-based subsetting would otherwise spend the read budget on control reads.

## Evidence

- [intro] A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs): "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [methods] Porechop adapter trimming is skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [readme] Porechop is a tool for finding and removing adapters from Oxford Nanopore reads.: [evidence span withheld — anchor status: not_found]
- [readme] Filtlong is a tool for filtering long reads by quality. It can take a set of long reads and produce a smaller, better subset.: [evidence span withheld — anchor status: external_doc]
