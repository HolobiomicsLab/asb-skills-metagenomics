---
name: ena-read-data-retrieval
description: 'Use when starting a large-scale metagenomic analysis from publicly archived
  sequencing runs: an ENA/SRA accession (BioProject, Biosample, Experiment, or Run)
  must be converted into local FASTQ files before downstream QC, alignment, or assembly
  steps.'
when_to_use_negative:
- Input is already local raw FASTQ data — proceed directly to fastp trimming/QC within
  the workflow instead of re-downloading
- Analysis targets long-read (Nanopore/PacBio) data — ARGprofiler is designed to only
  work with short-read sequencing data
- Only derived results (e.g. PanRes ARG abundance tables, mOTUs profiles) are needed
  and raw runs need not be reprocessed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: fastq-dl
  role: Downloads FASTQ files from ENA/SRA by accession, querying the ENA Data Warehouse
    API to resolve run metadata
  repo: https://github.com/rpetit3/fastq-dl
- name: ARGprofiler
  role: Downstream Snakemake workflow that consumes the downloaded raw reads for trimming,
    KMA alignment, ARGextender assembly, and Mash sketching
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:577672c4c96d3626085408846616751613b9561ed65a574ff311c6ae4bd01e09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rpetit3/fastq-dl
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - Mash
  - PanRes
  - fastp
  - fastq-dl
  - mOTUs
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# ena-read-data-retrieval

## Summary

Retrieve raw metagenomic FASTQ read files from the European Nucleotide Archive (ENA) or SRA using fastq-dl, as the data-acquisition entry point of the ARGprofiler resistome analysis workflow.

## When to use

Use when starting a large-scale metagenomic analysis from publicly archived sequencing runs: an ENA/SRA accession (BioProject, Biosample, Experiment, or Run) must be converted into local FASTQ files before downstream QC, alignment, or assembly steps.

## When NOT to use

- Input is already local raw FASTQ data — proceed directly to fastp trimming/QC within the workflow instead of re-downloading
- Analysis targets long-read (Nanopore/PacBio) data — ARGprofiler is designed to only work with short-read sequencing data
- Only derived results (e.g. PanRes ARG abundance tables, mOTUs profiles) are needed and raw runs need not be reprocessed

## Inputs

- ENA or SRA accession string (BioProject/Study, Biosample/Sample, Experiment, or Run), e.g. PRJEB34633
- ENA Data Warehouse REST API metadata (queried by fastq-dl)
- Target output directory (e.g. raw_reads/) for downloaded FASTQ files

## Outputs

- Raw FASTQ files (paired-end Illumina short reads) for each run in the accession set
- Downloaded read data organized in the raw_reads/ directory as consumed by the ARGprofiler Snakemake workflow

## How to apply

Query ENA via its Data Warehouse REST API using an accession (e.g. the Copenhagen sewage collection under PRJEB34633) with fastq-dl, which resolves metadata to locate and download the raw FASTQ files into a raw_reads directory. This step standardizes data provenance for the downstream Snakemake workflow: after download, ARGprofiler's rules run fastp trimming/QC, KMA alignment against mOTUs3 and PanRes, ARGextender targeted assembly, and Mash sketching (sketch size 1e4, k-mer size 31). Accession resolution means a study-level accession suffices to pull all associated runs, supporting the large-scale, uniform reprocessing goal of the pipeline. Verify downloads by checking that each run in the accession set has a matching FASTQ pair before launching the workflow, since ARGprofiler throughput is measured against total basepairs processed per sample.

## Related tools

- **fastq-dl** (Downloads FASTQ files from ENA/SRA by accession, querying the ENA Data Warehouse API to resolve run metadata) — https://github.com/rpetit3/fastq-dl
- **ARGprofiler** (Downstream Snakemake workflow that consumes the downloaded raw reads for trimming, KMA alignment, ARGextender assembly, and Mash sketching) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
fastq-dl --accession PRJEB34633 --source ena --outdir raw_reads/ --cpus 4
```

## Evaluation signals

- Each run identifier in the ENA accession set has a corresponding downloaded FASTQ (pair) in raw_reads/
- Downloaded reads are short-read (Illumina) FASTQ, compatible with fastp trimming/QC as the next workflow rule
- Subsequent ARGprofiler execution yields throughput metrics (basepairs per hour per sample) consistent with the raw data volume retrieved
- Retrieved dataset matches the intended study provenance (e.g. PRJEB34633 Copenhagen sewage runs) rather than unrelated accession resolution

## Limitations

- Retrieval depends on ENA API availability and correct accession resolution; a study-level accession pulls all runs, which may exceed intended scope
- fastq-dl covers ENA and SRA only — other archives require different acquisition tooling

## Evidence

- [readme] fastq-dl queries ENA via API for metadata: "takes an ENA/SRA accession (BioProject/Study, Biosample/Sample, Experiment, or Run) and queries ENA (via
[Data Warehouse API](https://www.ebi.ac.uk/ena/browse/search-rest)) to determine the"
- [other] Retrieve metagenomic reads from ENA via fastq-dl: [evidence span withheld — anchor status: not_found]
- [abstract] ARGprofiler is a Snakemake workflow supporting reuse of metagenomic data: "The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data"
- [discussion] Pipeline only works with short-read data: "ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies"
