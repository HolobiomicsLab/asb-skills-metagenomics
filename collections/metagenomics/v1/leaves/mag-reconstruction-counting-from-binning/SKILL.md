---
name: mag-reconstruction-counting-from-binning
description: Use when benchmarking metagenome assembly pipelines on simulated (e.g.
  CAMISIM) short and long reads and you need to quantify how hybrid assembly and co-assembly
  change N50, assembly size, and the number of recovered MAGs.
when_to_use_negative:
- Input data is not metagenomic read data (e.g. isolate genomes or already-assembled
  contigs) — the skill requires raw short and/or long reads to assemble and bin.
- The analysis goal is taxonomic classification or functional annotation of known
  MAGs rather than comparing assembly/binning completeness across pipeline configurations.
- Read-type gating is unknown or mixed (e.g. PacBio reads that would be wrongly subjected
  to Porechop adapter trimming) — correct gating must be established before the comparison
  is valid.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: nf-core/mag
  role: Pipeline executing preprocessing, hybrid assembly, and binning across the
    four assembly settings; source of assembly statistics and binning/MAG outputs
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine used to configure and launch the four nf-core/mag runs reproducibly
- name: Bowtie2
  role: Aligns short reads in the hybrid assembly setting
- name: minimap2
  role: Aligns long reads in the hybrid assembly setting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:2e48b68216a2ed16c3e3b7de98be42f41357a8877ecc8ed04ec5ea3bd453788c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - Nextflow
  - minimap2
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mag-reconstruction-counting-from-binning

## Summary

Run nf-core/mag (v2.1.0) across short-read/hybrid × sample-wise/co-assembly settings on CAMISIM-simulated reads, then extract assembly size, N50, and reconstructed MAG counts from the assembly statistics and binning outputs to benchmark assembly continuity and completeness.

## When to use

Use when benchmarking metagenome assembly pipelines on simulated (e.g. CAMISIM) short and long reads and you need to quantify how hybrid assembly and co-assembly change N50, assembly size, and the number of recovered MAGs.

## When NOT to use

- Input data is not metagenomic read data (e.g. isolate genomes or already-assembled contigs) — the skill requires raw short and/or long reads to assemble and bin.
- The analysis goal is taxonomic classification or functional annotation of known MAGs rather than comparing assembly/binning completeness across pipeline configurations.
- Read-type gating is unknown or mixed (e.g. PacBio reads that would be wrongly subjected to Porechop adapter trimming) — correct gating must be established before the comparison is valid.

## Inputs

- CAMISIM-simulated mouse-gut time-series paired-end short reads (FASTQ)
- CAMISIM-simulated long reads (PacBio/Nanopore FASTQ)
- nf-core/mag v2.1.0 samplesheet/config specifying sample-wise vs co-assembly grouping and short/long read paths

## Outputs

- Assembly statistics per setting (total assembly size, N50)
- Binning output with reconstructed MAGs per setting (MAG counts)
- mag_assembly_benchmark_table.csv comparing the four assembly settings

## How to apply

Download the CAMISIM-simulated mouse-gut time-series short and long reads (Zenodo DOI 10.5281/zenodo.5155395) and launch nf-core/mag v2.1.0 with Nextflow for four settings: short-read sample-wise, short-read co-assembly, and the two hybrid settings that align short reads with Bowtie2 and long reads with minimap2. Keep read-type gating of preprocessing steps intact — e.g. Porechop adapter trimming applies to Nanopore reads but is skipped for PacBio reads — and pool the correct samples into groups for the co-assembly runs. After each run, parse the assembly statistics for total assembly size and N50, and count the reconstructed MAGs from the binning output (the pipeline bins contigs using sample-wise group information and/or co-abundances). Tabulate the per-setting metrics into mag_assembly_benchmark_table.csv and compare the relative ordering of settings against Figure 2 of the paper.

## Related tools

- **nf-core/mag** (Pipeline executing preprocessing, hybrid assembly, and binning across the four assembly settings; source of assembly statistics and binning/MAG outputs) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine used to configure and launch the four nf-core/mag runs reproducibly)
- **Bowtie2** (Aligns short reads in the hybrid assembly setting)
- **minimap2** (Aligns long reads in the hybrid assembly setting)

## Examples

```
nextflow run nf-core/mag -r 2.1.0 -profile docker
```

## Evaluation signals

- Tabulated assembly size, N50, and MAG counts reproduce the reported trend: hybrid and co-assembly settings increase all three metrics relative to short-read-only sample-wise assembly.
- Each of the four settings produces a complete set of assembly statistics and a binning output with a countable MAG set.
- Co-assembly runs pool the intended sample groups; per-sample runs keep samples separate in the samplesheet.
- Any deviating setting triggers a re-check of read-type gating (Bowtie2 for short reads, minimap2 for long reads) and co-assembly grouping before the table is accepted.
- Porechop adapter trimming is confirmed skipped for PacBio long-read inputs.

## Limitations

- Results are demonstrated on CAMISIM-simulated data; real (non-simulated) time-series data may behave differently.
- The benchmark covers assembly size, N50, and MAG counts only; the pipeline version 2.1.0 lacked functional annotation, assembly refinement, and bin refinement steps.
- Only Muffin among compared pipelines supports hybrid assembly and none supports co-assembly, so cross-pipeline comparisons may be limited.

## Evidence

- [abstract] Hybrid and co-assembly improve assembly metrics: "combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [intro] Co-assembly increases depth and completeness: "co-assembly increases sequencing depth and can thus improve assembly completeness"
- [methods] Read-type-specific alignment in hybrid setting: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [discussion] Sample-wise group information for binning: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
- [supplementary] CAMISIM simulated benchmark dataset: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
