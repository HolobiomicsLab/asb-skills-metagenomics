---
name: mag-abundance-quantification
description: Use when you have assembled metagenomic contigs from multiple samples
  in a group and need MetaBAT2 binning plus a per-sample MAG abundance table, e.g.
  on CAMISIM simulated reads from Zenodo 10.5281/zenodo.5155395.
when_to_use_negative:
- 'Reads are long reads: alignment should be done with minimap2, not Bowtie2, because
  nf-core/mag gates alignment by read type'
- You only need assembly or taxonomic classification without binning and abundance
  estimation
- The dataset has a single sample with no group information, so no cross-sample co-abundance
  patterns can be computed
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: nf-core/mag
  role: Pipeline orchestrating binning and per-sample MAG abundance estimation
  repo: https://github.com/nf-core/mag
- name: Bowtie2
  role: Short-read alignment against contigs to compute per-contig sequencing depth
    for co-abundance
- name: MetaBAT2
  role: Bins contigs into MAGs using nucleotide frequencies and co-abundance across
    samples within the group
- name: Nextflow
  role: Workflow engine executing the nf-core/mag binning and abundance steps
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:e44ab1db994246c294f1dfcb35c00a9447614fd3f889dc9b8439028e5c56fd08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - MetaBAT2
  - Nextflow
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mag-abundance-quantification

## Summary

Quantifies per-sample abundances of metagenome assembled genomes (MAGs) produced by nf-core/mag, using per-sample read recruitment against assembled contigs (Bowtie2 for short reads, minimap2 for long reads) and MetaBAT2 co-abundance patterns within a sample group.

## When to use

Use when you have assembled metagenomic contigs from multiple samples in a group and need MetaBAT2 binning plus a per-sample MAG abundance table, e.g. on CAMISIM simulated reads from Zenodo 10.5281/zenodo.5155395.

## When NOT to use

- Reads are long reads: alignment should be done with minimap2, not Bowtie2, because nf-core/mag gates alignment by read type
- You only need assembly or taxonomic classification without binning and abundance estimation
- The dataset has a single sample with no group information, so no cross-sample co-abundance patterns can be computed

## Inputs

- CAMISIM simulated metagenomic short reads (Zenodo 10.5281/zenodo.5155395)
- Assembled contig FASTA files
- Sample group definition mapping samples to a user-specified group

## Outputs

- MAG bin FASTA files produced by MetaBAT2
- Per-sample MAG abundance table (one abundance value per MAG per sample)
- Per-contig mean sequencing depth per sample (Bowtie2 alignments of reads to contigs)

## How to apply

Index the assembled contigs with bowtie2-build, then map each sample's short reads back to the contigs with Bowtie2 (minimap2 is used instead when reads are long, since nf-core/mag gates alignment by read type) and compute the per-contig mean sequencing depth for every sample. Aggregate depths and sequence information across all samples of the user-specified group so that co-abundance patterns can be computed. Run MetaBAT2, which combines tetranucleotide nucleotide frequencies with the co-abundance distance to probabilistically cluster contigs into MAG bins. Finally estimate per-sample MAG abundance by summing read depth over the contigs of each bin normalized by bin length, yielding one abundance value per MAG per sample.

## Related tools

- **nf-core/mag** (Pipeline orchestrating binning and per-sample MAG abundance estimation) — https://github.com/nf-core/mag
- **Bowtie2** (Short-read alignment against contigs to compute per-contig sequencing depth for co-abundance)
- **MetaBAT2** (Bins contigs into MAGs using nucleotide frequencies and co-abundance across samples within the group)
- **Nextflow** (Workflow engine executing the nf-core/mag binning and abundance steps) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag -profile docker --input samplesheet.csv --outdir results
```

## Evaluation signals

- A non-empty set of MAG bin FASTA files is produced for every sample group
- A complete per-sample abundance table is generated, with one abundance value per MAG per sample in the group
- Per-contig depth values are available for every sample of the group before MetaBAT2 runs
- Matches the pipeline's automated nf-test assertions on the bundled test data
- Bins reflect co-abundance patterns computed within the same group, consistent with the article's group-wise binning description

## Limitations

- Bin refinement, functional annotation, and metatranscriptomic workflows were not yet integrated in nf-core/mag 2.1.0
- Group-wise co-abundance computation requires multiple samples assigned to the same user-specified group; single-sample groups give no co-abundance signal
- Abundances are derived from contig sequencing depths and are normalized only by bin length, so fragmented bins may affect estimates

## Evidence

- [discussion] Pipeline advantage: can use sample-wise group information for co-assembly and/or co-abundances for genome binning: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
- [abstract] nf-core/mag performs assembly, binning and taxonomic classification: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [methods] Short reads are aligned with Bowtie2, long reads with minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [supplementary] Exemplary results use CAMISIM simulated metagenomic data: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
- [discussion] Pipeline is maintained as part of nf-core with documentation at nf-co.re/mag: "It provides comprehensive usage documentation (https://nf-co.re/mag) and community support via a dedicated Slack channel"
