---
name: metagenome-assembly-benchmarking
description: Use when comparing hybrid vs. short-read-only assembly, or group-wise
  co-assembly vs. sample-wise assembly, using metrics like N50, assembly size, and
  MAG counts on simulated or real metagenomic time-series data.
when_to_use_negative:
- The analysis goal is taxonomic classification or binning quality alone rather than
  comparing assembly strategies — nf-core/mag supports those steps but this skill
  targets assembly metrics.
- Input data is a single-isolate genome assembly rather than metagenomic reads; hybrid-assembly
  and co-assembly comparisons are not meaningful there.
- Reads are already assembled contigs — re-running the four assembly settings requires
  raw FASTQ input.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: nf-core/mag
  role: Pipeline executing preprocessing, assembly, and binning for all four benchmark
    settings
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine used to configure and launch the nf-core/mag runs
  repo: https://github.com/nf-core/mag
- name: Bowtie2
  role: Aligns short reads in hybrid assembly settings
- name: minimap2
  role: Aligns long reads in hybrid assembly settings
- name: Porechop
  role: Adapter trimming for Oxford Nanopore long reads; skipped for PacBio reads
    (read-type gating)
  repo: rrwick/Porechop
- name: Filtlong
  role: Long-read quality filtering within the nf-core/mag preprocessing steps
  repo: rrwick/Filtlong
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:331e205359d34e02e803922c840d5bfdc8ed9d5e02648231434c4c9d509de341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - Filtlong
  - Nextflow
  - Porechop
  - minimap2
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metagenome-assembly-benchmarking

## Summary

Benchmark metagenome assembly strategies by running nf-core/mag v2.1.0 on CAMISIM-simulated reads under four settings (short/hybrid × sample-wise/co-assembly) and comparing assembly size, N50, and number of reconstructed MAGs.

## When to use

Use when comparing hybrid vs. short-read-only assembly, or group-wise co-assembly vs. sample-wise assembly, using metrics like N50, assembly size, and MAG counts on simulated or real metagenomic time-series data.

## When NOT to use

- The analysis goal is taxonomic classification or binning quality alone rather than comparing assembly strategies — nf-core/mag supports those steps but this skill targets assembly metrics.
- Input data is a single-isolate genome assembly rather than metagenomic reads; hybrid-assembly and co-assembly comparisons are not meaningful there.
- Reads are already assembled contigs — re-running the four assembly settings requires raw FASTQ input.

## Inputs

- CAMISIM-simulated mouse-gut time-series short reads (FASTQ)
- CAMISIM-simulated mouse-gut time-series long reads (FASTQ), from Zenodo DOI 10.5281/zenodo.5155395
- nf-core/mag v2.1.0 pipeline configuration (Nextflow params, sample sheet, co-assembly grouping)

## Outputs

- Per-setting assembly statistics (assembly size, N50) for the four assembly settings
- Per-setting counts of reconstructed MAGs from the binning output
- mag_assembly_benchmark_table.csv with assembly metrics per setting

## How to apply

Download the CAMISIM-simulated mouse-gut time-series reads (short and long) from Zenodo DOI 10.5281/zenodo.5155395. Configure and launch nf-core/mag v2.1.0 with Nextflow four times: (1) short-read sample-wise, (2) short-read co-assembly pooling samples for group-wise information, (3) hybrid sample-wise, and (4) hybrid co-assembly. In hybrid runs, align short reads with Bowtie2 and long reads with minimap2, keeping the read-type gating of preprocessing steps intact (e.g., Porechop adapter trimming is skipped for PacBio reads). From each run extract assembly size and N50 from the assembly statistics and count reconstructed MAGs from the binning output, then tabulate the per-setting metrics (e.g., mag_assembly_benchmark_table.csv) and compare the relative ordering of settings against the reported trend.

## Related tools

- **nf-core/mag** (Pipeline executing preprocessing, assembly, and binning for all four benchmark settings) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine used to configure and launch the nf-core/mag runs) — https://github.com/nf-core/mag
- **Bowtie2** (Aligns short reads in hybrid assembly settings)
- **minimap2** (Aligns long reads in hybrid assembly settings)
- **Porechop** (Adapter trimming for Oxford Nanopore long reads; skipped for PacBio reads (read-type gating)) — rrwick/Porechop
- **Filtlong** (Long-read quality filtering within the nf-core/mag preprocessing steps) — rrwick/Filtlong

## Examples

```
nextflow run nf-core/mag -r 2.1.0 --input samplesheet.csv --assembly coassemble --hybrid --bam短_read_aligner bowtie2
```

## Evaluation signals

- Tabulated assembly size, N50, and MAG counts reproduce the reported trend: hybrid assembly and group-wise co-assembly each increase assembly size, N50, and number of reconstructed MAGs versus the corresponding baseline.
- Relative ordering of the four settings matches the trend reported in Figure 2 of the article.
- Any setting that deviates is re-checked for correct read-type gating (Bowtie2 for short reads, minimap2 for long reads) and correct co-assembly grouping before the table is accepted.
- Each of the four nf-core/mag runs completes without mixing read types in preprocessing steps (e.g., Porechop must not run on PacBio reads).

## Limitations

- Results are demonstrated on CAMISIM-simulated metagenomic data; extrapolation to real microbiomes is not validated by this benchmark.
- Co-assembly and hybrid assembly increase resource demands and assembly size, which may not always translate to true completeness gains.
- The published nf-core/mag v2.1.0 lacks functional annotation, assembly refinement, and bin refinement steps, which could affect MAG recovery counts.

## Evidence

- [abstract] It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning.: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [methods] short- vs long-read alignment uses Bowtie2 vs minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [methods] Porechop adapter trimming is skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [intro] co-assembly increases sequencing depth and can thus improve assembly completeness: "co-assembly increases sequencing depth and can thus improve assembly completeness"
- [intro] Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost"
