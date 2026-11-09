---
name: co-assembly-sample-grouping
description: Use when multiple related metagenomic samples (e.g. a time series) should
  be assembled jointly per group rather than sample-wise, and assembly size, N50,
  or MAG counts are the target metrics.
when_to_use_negative:
- Samples are biologically unrelated and share no group information, so pooling would
  mix communities without depth benefit
- The analysis goal is per-sample genome-resolved profiles where sample-wise assembly
  (or binning using co-abundances) is required
- The input is already assembled contigs rather than raw FASTQ reads
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: nf-core/mag
  role: Pipeline executing the group-wise co-assembly and binning under Nextflow
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine running the nf-core/mag co-assembly setting
- name: Bowtie2
  role: Aligns short reads in the co-assembly read-type gating
- name: minimap2
  role: Aligns long reads in the hybrid co-assembly read-type gating
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:7ebd264682270c748530727bfb22b1497695480a27555dae8ddb158c1ff80da0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - Nextflow
  - minimap2
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# co-assembly-sample-grouping

## Summary

Pools metagenomic samples into user-defined groups for co-assembly with nf-core/mag, exploiting sample-wise group information to increase sequencing depth and improve assembly completeness and MAG recovery.

## When to use

Use when multiple related metagenomic samples (e.g. a time series) should be assembled jointly per group rather than sample-wise, and assembly size, N50, or MAG counts are the target metrics.

## When NOT to use

- Samples are biologically unrelated and share no group information, so pooling would mix communities without depth benefit
- The analysis goal is per-sample genome-resolved profiles where sample-wise assembly (or binning using co-abundances) is required
- The input is already assembled contigs rather than raw FASTQ reads

## Inputs

- Preprocessed short-read FASTQ files for the samples in each group (CAMISIM-simulated mouse-gut time-series reads, Zenodo 10.5281/zenodo.5155395)
- Optional long-read FASTQ files (Oxford Nanopore/PacBio) for hybrid co-assembly settings
- Sample-to-group mapping defining which samples are pooled for co-assembly
- nf-core/mag v2.1.0 configuration for Nextflow

## Outputs

- Group-wise co-assembled contigs per sample group
- Assembly statistics with assembly size and N50 per setting
- MAG binning output with counts of reconstructed MAGs
- Comparison table across assembly settings (e.g. mag_assembly_benchmark_table.csv)

## How to apply

Configure nf-core/mag (v2.1.0 in the benchmark) with Nextflow and supply a group definition that pools the samples belonging to the same biological group, so the assembler receives all reads of a group in one co-assembly run. Keep read-type gating intact: short reads are aligned with Bowtie2 and long reads with minimap2, and preprocessing steps such as Porechop adapter trimming (skipped for PacBio reads) are applied per read type. Run the co-assembly setting both for short-read-only and for hybrid (short+long) input to compare against the corresponding sample-wise assemblies. Extract assembly size, N50, and the number of reconstructed MAGs from the assembly statistics and binning output, and tabulate them (e.g. mag_assembly_benchmark_table.csv). The rationale is that co-assembly increases sequencing depth and can thus improve assembly completeness, while group-wise pooling preserves sample-wise group information rather than merging unrelated samples.

## Related tools

- **nf-core/mag** (Pipeline executing the group-wise co-assembly and binning under Nextflow) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine running the nf-core/mag co-assembly setting)
- **Bowtie2** (Aligns short reads in the co-assembly read-type gating)
- **minimap2** (Aligns long reads in the hybrid co-assembly read-type gating)

## Examples

```
nextflow run nf-core/mag -r 2.1.0 --input samplesheet.csv --coassemble_group_groupA 'sample1,sample2,sample3' --assembly_megahit -profile docker
```

## Evaluation signals

- Co-assembly should increase assembly size and N50 relative to the sample-wise assembly of the same group
- Number of reconstructed MAGs from the binning output should increase under group-wise co-assembly
- Read-type gating must hold: Bowtie2 used for short reads, minimap2 for long reads; Porechop skipped for PacBio reads
- Per-group runs must use the correct sample grouping; a deviating setting is re-checked for co-assembly grouping before the table is accepted
- Relative ordering of the four settings should reproduce the trend reported in Figure 2 of the CAMISIM time-series benchmark

## Limitations

- The pipeline lacks assembly and bin refinement steps, which may affect MAG quality estimates from co-assemblies
- Co-assembly requires justified sample grouping; only Muffin among existing pipelines supports hybrid assembly and none supports co-assembly, limiting external references
- Metrics depend on the simulated CAMISIM data (Zenodo 10.5281/zenodo.5155395); results may not transfer to real communities

## Evidence

- [intro] co-assembly increases sequencing depth and can thus improve assembly completeness: "co-assembly increases sequencing depth and can thus improve assembly completeness"
- [abstract] It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning.: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [discussion] it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
- [methods] short- vs long-read alignment uses Bowtie2 vs minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [supplementary] with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
