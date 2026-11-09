---
name: sra-read-data-retrieval
description: Use when a benchmark or reproduction task requires downloading raw short-read
  and long-read FASTQ data for specific samples identified by archive accessions (e.g.
  BioProject PRJDB4377, Supplementary Table S7).
when_to_use_negative:
- The analysis only needs the authors' published assemblies rather than raw reads
  — download the FigShare assembly bundle directly instead
- The input data are already local FASTQ/FASTA files and only the assembly step (MetaPlatanus)
  remains to be executed
- The target study provides no public accession or supplementary accession table for
  its raw reads (a noted missing signal for some mock/human gut benchmarks)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Downstream consumer of the retrieved paired-end (-IP1/-IP2) and ONT (-ont)
    FASTQ reads for contig-assembly, scaffolding, gap-closing and binning
  repo: https://github.com/rkajitani/MetaPlatanus
- name: minimap2
  role: Used downstream to map retrieved reads against assemblies (e.g. map-ont preset)
    supporting binning and evaluation of the downloaded data
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:86c7dade049851d285e7d57a05747d6e36c4fe985de589d2838afd240a4ca692
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - MEGAHIT
  - MetaPlatanus
  - NextPolish
  - OPERA-MS
  - Racon
  - TGS-GapCloser
  - metaFlye
  - minimap2
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sra-read-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves raw sequencing reads (Illumina paired-end and ONT/PacBio long reads) from public sequence read archives such as DDBJ SRA, using BioProject accessions and supplementary accession tables, to serve as the input reads for de novo metagenome assembly benchmarks like the MetaPlatanus saliva PromethION study.

## When to use

Use when a benchmark or reproduction task requires downloading raw short-read and long-read FASTQ data for specific samples identified by archive accessions (e.g. BioProject PRJDB4377, Supplementary Table S7).

## When NOT to use

- The analysis only needs the authors' published assemblies rather than raw reads — download the FigShare assembly bundle directly instead
- The input data are already local FASTQ/FASTA files and only the assembly step (MetaPlatanus) remains to be executed
- The target study provides no public accession or supplementary accession table for its raw reads (a noted missing signal for some mock/human gut benchmarks)

## Inputs

- BioProject or SRA/DDBJ run accession list (e.g. PRJDB4377, Supplementary Table S7 accessions)
- Supplementary table mapping samples to read archives and sequencing depths (28-75 Gbp PromethION, Illumina paired-end)
- Optional FigShare assembly bundle doi:10.6084/m9.figshare.15090603.v1 for cross-checking

## Outputs

- Per-sample Illumina paired-end FASTQ files (PE_1.fq, PE_2.fq)
- Per-sample ONT PromethION long-read FASTQ files (ONT.fq)
- Provenance table linking sample IDs to archive accessions and library types

## How to apply

Identify the sample-to-accession mapping from the study's supplementary tables (e.g. Supplementary Table S7 lists the four human saliva samples sequenced with 28-75 Gbp ONT PromethION reads plus Illumina paired ends; the MP mock samples are under DDBJ BioProject PRJDB4377). Query the archive (DDBJ SRA or its ENA/NCBI mirrors) by BioProject or run accession to obtain FASTQ files for each library type: Illumina paired-end FASTQ for short reads and ONT FASTQ for long reads. Keep library-type provenance per sample, since downstream assembly passes paired-end reads through MetaPlatanus -IP1/-IP2 and long reads through -ont. Optionally fetch the authors' precomputed assemblies from FigShare (doi:10.6084/m9.figshare.15090603.v1) to validate downloaded-read reproductions against published outputs. Verify integrity by matching read counts/base volumes (e.g. Gbp scale) to the reported quantities before assembly.

## Related tools

- **MetaPlatanus** (Downstream consumer of the retrieved paired-end (-IP1/-IP2) and ONT (-ont) FASTQ reads for contig-assembly, scaffolding, gap-closing and binning) — https://github.com/rkajitani/MetaPlatanus
- **minimap2** (Used downstream to map retrieved reads against assemblies (e.g. map-ont preset) supporting binning and evaluation of the downloaded data) — https://github.com/lh3/minimap2

## Examples

```
prefetch --option-file sra_accessions.txt && fasterq-dump --split-files --outdir reads/ $(cat sra_accessions.txt)
```

## Evaluation signals

- Every sample listed in Supplementary Table S7 yields both an Illumina paired-end pair and an ONT FASTQ file with no missing libraries
- Retrieved base volumes are consistent with the reported PromethION depths of 28-75 Gbp per saliva sample
- Mock-sample reads resolve under the stated BioProject PRJDB4377 in DDBJ SRA
- Assemblies produced from the downloaded reads reproduce the published result: 22 near-complete scaffolds assembled exclusively by MetaPlatanus (Supplementary Tables S22, S24, S25)
- Read files pass FASTQ sanity checks (consistent read lengths, no truncation after download/checksum verification)

## Limitations

- No dataset accessions or availability statements were available in the article body for the mock or human gut benchmark data, so retrieval relies on supplementary tables
- MetaPlatanus requires at least one short-read paired-end library, so long-read-only archive entries cannot be used with this pipeline
- Reproducing large PromethION datasets (tens of Gbp per sample) entails substantial download volume and storage

## Evidence

- [other] Retrieve the raw Illumina paired-end and ONT PromethION read sets for the four human saliva samples from the public sequence read archive (Supplementary Table S7).: [evidence span withheld — anchor status: not_found]
