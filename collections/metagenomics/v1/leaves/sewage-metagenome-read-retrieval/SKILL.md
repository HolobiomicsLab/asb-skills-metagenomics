---
name: sewage-metagenome-read-retrieval
description: Use when a benchmark or pipeline needs raw short-read FASTQ files for
  defined sewage/urban metagenome BioProject accessions (e.g. PRJEB34633, PRJEB63576)
  retrieved reproducibly from ENA.
when_to_use_negative:
- The input is already local FASTQ files and the task is QC or sketching rather than
  data acquisition (use fastp or Mash instead).
- The target data are long-read (PacBio/Nanopore) sequencing runs — the ARGprofiler
  benchmark context is short-read Illumina data, which fastq-dl retrieval here is
  scoped to.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: fastq-dl
  role: Downloads FASTQ files from ENA/SRA by accession (BioProject/Study/Sample/Experiment/Run),
    querying ENA metadata via the Data Warehouse API
  repo: https://github.com/rpetit3/fastq-dl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:6023eaf394a56a24f55bc2f1e1ac0b5104e410431282409ba454b99ec30f7ad2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rpetit3/fastq-dl
  tools:
  - ARGprofiler
  - CCPhylo
  - Mash
  - fastq-dl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Sewage metagenome read retrieval from ENA

## Summary

Retrieves short-read FASTQ data for sewage metagenomic samples from the European Nucleotide Archive using fastq-dl, e.g. the 36 Copenhagen sewage samples (PRJEB34633), 18 Munk et al. 2022 urban sewage samples, and 18 technical replicate resequencings of one Copenhagen sample (PRJEB63576) used to benchmark Mash sketch parameters in ARGprofiler.

## When to use

Use when a benchmark or pipeline needs raw short-read FASTQ files for defined sewage/urban metagenome BioProject accessions (e.g. PRJEB34633, PRJEB63576) retrieved reproducibly from ENA.

## When NOT to use

- The input is already local FASTQ files and the task is QC or sketching rather than data acquisition (use fastp or Mash instead).
- The target data are long-read (PacBio/Nanopore) sequencing runs — the ARGprofiler benchmark context is short-read Illumina data, which fastq-dl retrieval here is scoped to.

## Inputs

- ENA BioProject/Study accessions (e.g. PRJEB34633, PRJEB63576)
- List of target sample/run accessions with metadata from the ENA Data Warehouse API
- Output directory for downloaded FASTQ files

## Outputs

- Compressed FASTQ files (paired-end R1/R2) for 72 sewage metagenomic sequencing runs
- ENA metadata (run/sample/experiment mappings) resolved by fastq-dl for provenance

## How to apply

Query ENA for each target study accession (BioProject/Study level works with fastq-dl, which resolves metadata via the ENA Data Warehouse API) and download the associated FASTQ files with fastq-dl. Assemble the sample set deliberately: for the Mash sketch parameter benchmark this meant 36 Copenhagen sewage samples from PRJEB34633, 18 urban sewage samples from Munk et al. 2022, and 18 technical replicate resequencings of one Copenhagen sample from PRJEB63576 — the replicate resequencings are essential because they are the within-sample signal used to judge parameter choices downstream. Verify that all 72 sequencing runs are present after download and that paired-end files are complete before passing them to sketching (Mash 2.3) or quality control (fastp).

## Related tools

- **fastq-dl** (Downloads FASTQ files from ENA/SRA by accession (BioProject/Study/Sample/Experiment/Run), querying ENA metadata via the Data Warehouse API) — https://github.com/rpetit3/fastq-dl

## Examples

```
fastq-dl --accession PRJEB63576 --outdir reads --cpus 8
```

## Evaluation signals

- All 72 expected sequencing runs (36 Copenhagen + 18 urban sewage + 18 technical replicate resequencings) are present with non-empty FASTQ files.
- Paired-end downloads are complete: R1/R2 file pairs exist and read counts match per run.
- fastq-dl reports successful metadata resolution against ENA for each accession (no failed or skipped runs).
- The replicate resequencing runs from PRJEB63576 are correctly attributed to their parent Copenhagen sample so downstream Mash distance grouping is valid.

## Limitations

- Retrieval depends on ENA availability and complete submission metadata; runs missing or embargoed at the archive cannot be downloaded.
- Accessions must be correct at the study level — mixing up PRJEB63576 (replicate resequencings) with PRJEB34633 (original Copenhagen samples) breaks the within- vs between-sample distance analysis.

## Evidence

- [methods] Retrieve the short-read sequencing data for the 72 sewage metagenomes (36 Copenhagen sewage samples from PRJEB34633, 18 samples from Munk et al. 2022 urban sewage projects, and 18 technical replicate resequencings of one Copenhagen sample from PRJEB63576) from ENA using fastq-dl.: [evidence span withheld — anchor status: not_found]
- [readme] fastq-dl takes an ENA/SRA accession and queries ENA via the Data Warehouse API to determine associated metadata.: "`fastq-dl` takes an ENA/SRA accession (BioProject/Study, Biosample/Sample, Experiment, or Run) and queries ENA (via
[Data Warehouse API](https://www.ebi.ac.uk/ena/browse/search-rest)) to determine"
- [readme] fastq-dl downloads FASTQ files from the European Nucleotide Archive or the Sequence Read Archive repositories.: "Download FASTQ files from the [European Nucleotide Archive](https://www.ebi.ac.uk/ena) or the
[Sequence Read Archive](https://www.ncbi.nlm.nih.gov/sra) repositories."
