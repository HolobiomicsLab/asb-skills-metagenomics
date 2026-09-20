---
name: co-abundance-genome-binning
description: Use when you have assembled metagenomic contigs plus short reads from
  multiple samples grouped by the user and need to bin contigs into MAGs exploiting
  sample-wise co-abundance, or to estimate per-sample MAG abundances.
when_to_use_negative:
- Input is raw reads without an assembly — this skill bins assembled contigs, it does
  not assemble them (nf-core/mag runs assembly upstream)
- Only a single sample with no group information is available, so no cross-sample
  co-abundance signal can be computed
- The goal is bin refinement or functional annotation — the article lists these as
  future additions, not part of this binning step
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: nf-core/mag
  role: Pipeline that orchestrates read mapping, depth computation, group-wise co-abundance
    calculation, MetaBAT2 binning and per-sample MAG abundance estimation
  repo: https://github.com/nf-core/mag
- name: Bowtie2
  role: Maps each sample's short reads to the assembled contigs to derive per-contig
    sequencing depths used for co-abundance
- name: MetaBAT2
  role: Bins contigs into MAGs using nucleotide (tetranucleotide) frequencies combined
    with co-abundance distance
- name: Nextflow
  role: Workflow engine in which nf-core/mag is written, providing reproducible execution
    of the binning workflow
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:c34f93192ff9acfa1c18a6ae20fbd25c13a370fdbc9cc078bd5a088a5c4117b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - MetaBAT2
  - Nextflow
  - nf-core/mag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# co-abundance-genome-binning

## Summary

Reconstruct metagenome assembled genomes (MAGs) by clustering assembled contigs with MetaBAT2, combining tetranucleotide nucleotide frequencies with co-abundance patterns computed across samples in the same group, as implemented in the nf-core/mag pipeline.

## When to use

Use when you have assembled metagenomic contigs plus short reads from multiple samples grouped by the user and need to bin contigs into MAGs exploiting sample-wise co-abundance, or to estimate per-sample MAG abundances.

## When NOT to use

- Input is raw reads without an assembly — this skill bins assembled contigs, it does not assemble them (nf-core/mag runs assembly upstream)
- Only a single sample with no group information is available, so no cross-sample co-abundance signal can be computed
- The goal is bin refinement or functional annotation — the article lists these as future additions, not part of this binning step

## Inputs

- Assembled contigs (FASTA) from metagenome assembly
- Preprocessed short reads (FASTQ) from each sample in the group
- Sample-to-group mapping (nf-core/mag grouping information)
- Bowtie2 index of the contigs (produced by bowtie2-build)

## Outputs

- MAG bin FASTA files produced by MetaBAT2
- Per-contig mean sequencing depth table per sample
- Per-sample MAG abundance table (abundance per MAG per sample, depth summed over bin contigs normalized by bin length)

## How to apply

Index the assembled contigs with bowtie2-build and map each sample's short reads to the contigs with Bowtie2 (nf-core/mag uses Bowtie2 for short reads and minimap2 for long reads), computing the per-contig mean sequencing depth for every sample. Aggregate depth and sequence information across all samples belonging to the user-specified group so that co-abundance patterns can be computed. Run MetaBAT2 on the contigs supplying per-contig depths and co-abundances; MetaBAT2 probabilistically clusters contigs into MAG bins using tetranucleotide frequencies together with the co-abundance distance. Finally estimate per-sample MAG abundance by summing read depth over the contigs of each bin, normalized by bin length, yielding an abundance value per MAG per sample. Rationale: nf-core/mag explicitly utilizes sample-wise group information to compute co-abundances for the genome binning step.

## Related tools

- **nf-core/mag** (Pipeline that orchestrates read mapping, depth computation, group-wise co-abundance calculation, MetaBAT2 binning and per-sample MAG abundance estimation) — https://github.com/nf-core/mag
- **Bowtie2** (Maps each sample's short reads to the assembled contigs to derive per-contig sequencing depths used for co-abundance)
- **MetaBAT2** (Bins contigs into MAGs using nucleotide (tetranucleotide) frequencies combined with co-abundance distance)
- **Nextflow** (Workflow engine in which nf-core/mag is written, providing reproducible execution of the binning workflow) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag --input samplesheet.csv --outdir results -profile docker
```

## Evaluation signals

- A non-empty set of MAG bin FASTA files is produced for the group
- A complete per-sample abundance table is produced with one abundance value per MAG per sample in the group
- The nf-test assertions on the bundled CAMISIM test data (Zenodo 10.5281/zenodo.5155395) pass
- Per-contig depth tables exist for every sample before binning, confirming Bowtie2 mapping completed for all samples
- Bins are consistent with MetaBAT2 usage of tetranucleotide frequencies plus co-abundance distance (bins reflect group-wise co-abundance, not single-sample coverage alone)

## Limitations

- Co-abundance binning by default uses only samples within the same user-specified group; samples outside the group do not contribute signal
- Bin refinement was not included in the presented nf-core/mag version 2.1.0 — the authors list it as a future extension, so bin quality must be assessed separately
- Alignment backend is gated by read type: Bowtie2 for short reads, minimap2 for long reads; mixing them up is invalid

## Evidence

- [supplementary] nf-core/mag bins contigs into individual MAGs using MetaBAT2 based on nucleotide frequencies and co-abundance patterns across samples (within the same group by default): [evidence span withheld — anchor status: not_found]
- [discussion] it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
- [abstract] It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning.: "utilize sample-wise group-information for co-assembly and genome binning"
- [methods] short- vs long-read alignment uses Bowtie2 vs minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [supplementary] with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
