---
name: containerized-bioinformatics-execution
description: Use when the goal is to execute the CAMAMED pipeline (taxonomic/functional
  profiling of metagenomic FASTQ data) without manually installing its dependencies
  — i.e., when Docker is available and the camamed Docker images can be pulled from
  hub.docker.com.
when_to_use_negative:
- Input is already a processed abundance/feature table (e.g., taxa or gene counts)
  — in that case the downstream CSS normalization or metagenomeSeq analysis applies
  directly and containerized raw-read execution is unnecessary
- The user must use tool versions other than those frozen in the image (e.g., a newer
  FastQC or SRA Toolkit), since the container pre-packages specific versions and the
  README gives no per-tool version configuration
- Docker is unavailable or prohibited in the execution environment; the README offers
  no non-containerized dependency installation path besides the manual
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: CAMAMED
  role: The pipeline whose full toolchain is executed inside the container; initialized
    with camamed_init and configured per the CAMAMED-manual.pdf
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Container runtime; the ready-to-use images camamed_pipeline (without MetaPhlAn2
    databases) and camamed_pipeline_db (with MetaPhlAn2 databases) eliminate all dependency
    installation
- name: SRA Toolkit
  role: fastq-dump (v2.8.2) pre-packaged in the image converts SRA archives to FASTQ/FASTA
    as the first pipeline stage
- name: FastQC
  role: FastQC v0.11.5, pre-installed in the container, generates per-sample quality-control
    reports on the converted reads
- name: SeqKit
  role: SeqKit v0.10.1, pre-installed in the container, computes sequence statistics
    consolidated into seqkit_stats.tsv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:ed23f3c8a6045befbfc284319918acdd1e9dafa519cf4f530265052a6b66dda0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - FastQC
  - SRA Toolkit
  - SeqKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# containerized-bioinformatics-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the CAMAMED composition-aware metagenomic analysis pipeline from its prebuilt Docker images (camamed_pipeline or camamed_pipeline_db), so that all bundled tools (SRA Toolkit, FastQC, SeqKit, mapping and normalization components) execute with correct dependencies without local installation.

## When to use

Use when the goal is to execute the CAMAMED pipeline (taxonomic/functional profiling of metagenomic FASTQ data) without manually installing its dependencies — i.e., when Docker is available and the camamed Docker images can be pulled from hub.docker.com.

## When NOT to use

- Input is already a processed abundance/feature table (e.g., taxa or gene counts) — in that case the downstream CSS normalization or metagenomeSeq analysis applies directly and containerized raw-read execution is unnecessary
- The user must use tool versions other than those frozen in the image (e.g., a newer FastQC or SRA Toolkit), since the container pre-packages specific versions and the README gives no per-tool version configuration
- Docker is unavailable or prohibited in the execution environment; the README offers no non-containerized dependency installation path besides the manual

## Inputs

- Metagenomic sample archives in SRA format (converted inside the container to FASTQ/FASTA via fastq-dump)
- Host-mapped gene catalog reference data (non-redundant gene catalogs) and KEGG mapping files
- Docker image camamed_pipeline or camamed_pipeline_db from hub.docker.com (camamed_pipeline_db bundles MetaPhlAn2 databases)

## Outputs

- Container-executed CAMAMED pipeline outputs: taxon, gene, KO, EC number and reaction abundance/annotation tables
- Per-sample FastQC HTML quality-control reports (per-base quality, GC content, adapter contamination)
- SeqKit statistics table seqkit_stats.tsv (read counts, min/mean/max sequence lengths, GC content)

## How to apply

First initialize the pipeline with `python camamed_init` as instructed in the CAMAMED manual, then run the pipeline inside one of the two published Docker images. Choose `camamed_pipeline_db` when MetaPhlAn2 databases are needed and not present locally; choose `camamed_pipeline` when the databases are excluded or supplied separately. The image pre-packages every tool in the chain — fastq-dump (SRA Toolkit 2.8.2) for SRA→FASTQ conversion, FastQC v0.11.5 for per-base quality/GC/adapter reports, and SeqKit v0.10.1 for read-count and length statistics — so the user avoids version-mismatch and installation obstacles. Because the container fixes the tool versions, results (e.g., the seqkit_stats.tsv statistics table, FastQC HTML reports, taxon/gene/KO/EC/reaction annotation levels) are reproducible across runs. The README explicitly states there is no need to install any software dependencies when the Docker image is used.

## Related tools

- **CAMAMED** (The pipeline whose full toolchain is executed inside the container; initialized with camamed_init and configured per the CAMAMED-manual.pdf) — https://github.com/mhnb/camamed
- **Docker** (Container runtime; the ready-to-use images camamed_pipeline (without MetaPhlAn2 databases) and camamed_pipeline_db (with MetaPhlAn2 databases) eliminate all dependency installation)
- **SRA Toolkit** (fastq-dump (v2.8.2) pre-packaged in the image converts SRA archives to FASTQ/FASTA as the first pipeline stage)
- **FastQC** (FastQC v0.11.5, pre-installed in the container, generates per-sample quality-control reports on the converted reads)
- **SeqKit** (SeqKit v0.10.1, pre-installed in the container, computes sequence statistics consolidated into seqkit_stats.tsv)

## Examples

```
python camamed_init  # then execute the pipeline inside the pulled image, e.g. via docker run on camamed/camamed_pipeline_db from hub.docker.com
```

## Evaluation signals

- Container pulls succeed for the named images and `python camamed_init` completes without dependency errors
- No tool is installed manually — verify the run used the image's pre-packaged versions (SRA Toolkit 2.8.2, FastQC v0.11.5, SeqKit v0.10.1)
- Per-sample FastQC HTML reports exist and flag per-base quality, GC content and adapter contamination as expected for the converted FASTQ files
- seqkit_stats.tsv is produced as a tab-separated table with read counts and min/mean/max sequence lengths per sample
- Pipeline outputs cover all five annotation levels (taxon, gene, KO, EC number, reaction); the camamed_pipeline image without databases should fail or skip MetaPhlAn2-based taxonomic steps unless databases are supplied

## Limitations

- The camamed_pipeline image ships without MetaPhlAn2 databases; only camamed_pipeline_db includes them, so image choice affects taxonomic profiling out of the box
- The README states no software versions for CAMAMED or its components and provides no changelog, so version drift inside images is hard to audit from documentation alone
- Non-Docker installation requires following CAMAMED-manual.pdf manually; the pipeline otherwise leaves the nonprofessional user exposed to software installation obstacles

## Evidence
