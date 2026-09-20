---
name: ena-dataset-retrieval
description: Use when the analysis requires raw short-read FASTQ data for a published
  study identified only by ENA/SRA accessions (BioProject/Study, e.g. PRJEB40798,
  ERP015409) and no local FASTQ files exist yet.
when_to_use_negative:
- The FASTQ reads are already available locally or on shared storage — re-downloading
  wastes bandwidth and API load
- The downstream tool requires long-read data — ARGprofiler is designed to only work
  with short-read sequencing data
- The task only needs summary statistics already published in the paper — retrieval
  and re-running the pipeline is unnecessary
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: fastq-dl
  role: Downloads FASTQ files from ENA/SRA by resolving the accession through the
    ENA Data Warehouse API
  repo: https://github.com/rpetit3/fastq-dl
- name: ARGprofiler
  role: Downstream Snakemake pipeline that consumes the retrieved FASTQ reads for
    ARG abundance and flanking-region analysis
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:89e8ef6782d6b176ec48838ccf4c183aceb523cce9bbd52db27c0aa564b761a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rpetit3/fastq-dl
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - fastq-dl
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ena-dataset-retrieval

## Summary

Programmatically download public metagenomic FASTQ reads (and associated metadata) from ENA/SRA accessions so that published studies can be re-analyzed with ARGprofiler, ARGextender, and related resistome workflows.

## When to use

Use when the analysis requires raw short-read FASTQ data for a published study identified only by ENA/SRA accessions (BioProject/Study, e.g. PRJEB40798, ERP015409) and no local FASTQ files exist yet.

## When NOT to use

- The FASTQ reads are already available locally or on shared storage — re-downloading wastes bandwidth and API load
- The downstream tool requires long-read data — ARGprofiler is designed to only work with short-read sequencing data
- The task only needs summary statistics already published in the paper — retrieval and re-running the pipeline is unnecessary

## Inputs

- ENA or SRA accession (BioProject/Study, Biosample/Sample, Experiment, or Run) — e.g. PRJEB40798, PRJEB40816, PRJEB40815, PRJEB27621, ERP015409
- Run-level metadata (library layout, instrument) from the ENA Data Warehouse API

## Outputs

- FASTQ files (one or paired pair per sequencing run) downloaded to a local directory
- Run/sample metadata resolved from ENA for provenance and bookkeeping
- Optionally, downloaded de novo assemblies when only scaffolds are needed

## How to apply

Take the study-level accession (e.g. BioProject IDs such as PRJEB40798, PRJEB40816, PRJEB40815, PRJEB27621, or the study accession ERP015409) and use fastq-dl, which queries the ENA Data Warehouse API to resolve the accession into run-level metadata, then downloads the associated FASTQ files into a working directory. Once FASTQ files are on disk, feed them into downstream ARGprofiler steps (fastp QC, KMA-based gene detection against PanRes, ARGextender targeted de novo assembly with SPAdes). This bulk-retrieval pattern underpins the ARGprofiler goal of enabling reuse of public metagenomic sequences for global epidemiological resistome analyses. Decide between downloading raw reads (needed for ARGextender extension) versus already-published de novo assemblies when the downstream task only requires scaffolds.

## Related tools

- **fastq-dl** (Downloads FASTQ files from ENA/SRA by resolving the accession through the ENA Data Warehouse API) — https://github.com/rpetit3/fastq-dl
- **ARGprofiler** (Downstream Snakemake pipeline that consumes the retrieved FASTQ reads for ARG abundance and flanking-region analysis) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
fastq-dl --accession PRJEB40798 --outdir ena_reads
```

## Evaluation signals

- All runs under the requested accession are present on disk (file count matches ENA run metadata for the study, e.g. the 951 urban sewage samples)
- Each FASTQ file is non-empty and passes FASTQ integrity checks (consistent read lengths, no truncation) before being fed to KMA/SPAdes
- Run/sample identifiers in downloaded filenames match the ENA metadata table returned for the accession
- Downstream ARGextender/KMA outputs are produced for every sample without missing-input failures in the Snakemake DAG
- Downloaded data reproduce published benchmark figures when re-analyzed (e.g. mean 57 distinct ARGs detected per sewage sample with ARGextender)

## Limitations

- Retrieval by accession yields whatever the original submission contains; quality control and layout assumptions of the original study are inherited
- Large-scale downloads of hundreds to thousands of runs (e.g. 951 samples) can be slow and require disk planning
- Only public ENA/SRA content can be retrieved; controlled-access datasets are not covered

## Evidence

- [readme] fastq-dl takes an ENA/SRA accession (BioProject/Study, Biosample/Sample, Experiment, or Run) and queries ENA (via Data Warehouse API) to determine the associated metadata.: "fastq-dl takes an ENA/SRA accession (BioProject/Study, Biosample/Sample, Experiment, or Run) and queries ENA (via
[Data Warehouse API](https://www.ebi.ac.uk/ena/browse/search-rest)) to determine the"
- [intro] ARGprofiler is another step toward enabling the reuse of metagenomic sequences: "which allows any user to process sequencing data to perform epidemiological analyses of ARGs globally. ARGprofiler is another step toward enabling the reuse of metagenomic sequences"
- [abstract] the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs: "the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs"
