---
name: assembly-continuity-metric-tabulation
description: Use when comparing contiguity and binning yield of metagenome assemblies
  produced under different read-type (short vs hybrid) or grouping (sample-wise vs
  co-assembly) settings, using CAMISIM-style simulated time-series reads.
when_to_use_negative:
- Data are real (not simulated) reads with a reference genome available — assembly
  quality should then be validated against the reference rather than by continuity
  metrics alone
- Only a single assembly setting is being run, so there is no cross-setting comparison
  to tabulate
- The pipeline version differs substantially from nf-core/mag 2.1.0 and lacks the
  read-type-gated hybrid/co-assembly configuration described here
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: nf-core/mag
  role: Pipeline executing the four assembly settings (sample-wise/co-assembly, short-read/hybrid),
    assembly statistics, and binning
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine used to launch and reproduce the nf-core/mag runs
  repo: https://github.com/nf-core/mag
- name: Bowtie2
  role: Aligns short reads in the hybrid assembly settings
- name: minimap2
  role: Aligns long reads in the hybrid assembly settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:abae90b182012a336225e4f5c67f1c598ebf5be5ddf1012218575f9ab09113d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - Nextflow
  - minimap2
  - nf-core/mag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# assembly-continuity-metric-tabulation

## Summary

Tabulates assembly size, N50, and number of reconstructed MAGs across four nf-core/mag v2.1.0 assembly settings (short-read sample-wise, short-read co-assembly, and their hybrid counterparts) on CAMISIM-simulated metagenomic data, to quantify how hybrid assembly and group-wise co-assembly improve assembly continuity and MAG recovery.

## When to use

Use when comparing contiguity and binning yield of metagenome assemblies produced under different read-type (short vs hybrid) or grouping (sample-wise vs co-assembly) settings, using CAMISIM-style simulated time-series reads.

## When NOT to use

- Data are real (not simulated) reads with a reference genome available — assembly quality should then be validated against the reference rather than by continuity metrics alone
- Only a single assembly setting is being run, so there is no cross-setting comparison to tabulate
- The pipeline version differs substantially from nf-core/mag 2.1.0 and lacks the read-type-gated hybrid/co-assembly configuration described here

## Inputs

- CAMISIM-simulated mouse-gut time-series short reads (FASTQ, Zenodo 10.5281/zenodo.5155395)
- CAMISIM-simulated mouse-gut time-series long reads (FASTQ, Zenodo 10.5281/zenodo.5155395)
- nf-core/mag v2.1.0 assembly statistics output per run
- nf-core/mag v2.1.0 binning output per run

## Outputs

- mag_assembly_benchmark_table.csv containing assembly size, N50, and reconstructed-MAG counts for the four assembly settings

## How to apply

Download the CAMISIM-simulated mouse-gut time-series short and long reads from Zenodo DOI 10.5281/zenodo.5155395. Run nf-core/mag v2.1.0 under Nextflow four times: short-read sample-wise assembly, short-read co-assembly pooling samples into groups, and the two hybrid settings in which short reads are aligned with Bowtie2 and long reads with minimap2, keeping read-type gating of preprocessing steps intact (e.g. Porechop adapter trimming skipped for PacBio reads). For each run, extract assembly size and N50 from the assembly statistics output and count reconstructed MAGs from the binning output. Assemble the per-setting values into mag_assembly_benchmark_table.csv and compare the relative ordering of the four settings against the trend reported in Figure 2 of the source article. The rationale is that hybrid assembly combines short- and long-read advantages for continuity, while co-assembly increases sequencing depth and can improve assembly completeness.

## Related tools

- **nf-core/mag** (Pipeline executing the four assembly settings (sample-wise/co-assembly, short-read/hybrid), assembly statistics, and binning) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine used to launch and reproduce the nf-core/mag runs) — https://github.com/nf-core/mag
- **Bowtie2** (Aligns short reads in the hybrid assembly settings)
- **minimap2** (Aligns long reads in the hybrid assembly settings)

## Examples

```
nextflow run nf-core/mag -r 2.1.0 --input samplesheet.csv --outdir results -profile docker
```

## Evaluation signals

- Tabulated assembly size, N50, and MAG counts reproduce the reported trend: hybrid and co-assembly settings improve assembly continuity and MAG recovery relative to short-read sample-wise assembly
- Each of the four settings appears as a row in mag_assembly_benchmark_table.csv with all three metrics populated
- Any setting deviating from the expected ordering is re-checked for correct read-type gating of preprocessing steps and correct co-assembly grouping before the table is accepted
- Each run was launched with nf-core/mag v2.1.0 via Nextflow, and hybrid runs used Bowtie2 for short reads and minimap2 for long reads
- Input reads trace back to Zenodo DOI 10.5281/zenodo.5155395

## Limitations

- Results are based on CAMISIM-simulated metagenomic data, so absolute metric values may not transfer to real communities
- Porechop adapter trimming is skipped for PacBio reads; incorrect read-type gating is the first thing to check if a setting deviates from the expected trend
- nf-core/mag 2.1.0 lacks functional annotation, bin refinement, and assembly refinement steps, so additional quality dimensions (e.g. bin completeness/contamination refinement) are outside this comparison

## Evidence

- [abstract] It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning.: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [intro] Hybrid assembly combines advantages of both read types: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies"
- [intro] co-assembly increases sequencing depth and can thus improve assembly completeness: "co-assembly increases sequencing depth and can thus improve assembly completeness"
- [methods] short- vs long-read alignment uses Bowtie2 vs minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [methods] Porechop adapter trimming is skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [supplementary] CAMISIM simulated data availability: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
