---
name: metagenomic-contig-depth-estimation
description: Use when you have assembled metagenomic contigs plus per-sample short-read
  FASTQs and need per-contig mean sequencing depths across samples to feed co-abundance-based
  binning (e.g., MetaBAT2) or to estimate MAG abundances.
when_to_use_negative:
- Input is long reads only — nf-core/mag routes long-read alignment to minimap2, not
  Bowtie2, so the Bowtie2 short-read depth workflow does not apply
- No multi-sample or group information exists and co-abundance-based binning is not
  the goal — single-sample depth may not support co-abundance clustering
- Contigs have not yet been produced (raw reads only) — assembly must precede depth
  estimation
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: nf-core/mag
  role: Pipeline orchestrating the mapping, per-contig depth computation, group-wise
    aggregation, and handoff of depths/co-abundances to MetaBAT2 for binning
  repo: https://github.com/nf-core/mag
- name: Bowtie2
  role: Aligns each sample's short reads to the indexed contigs; bowtie2-build indexes
    contigs before mapping
- name: MetaBAT2
  role: Consumes the per-contig depths and co-abundances (with nucleotide frequencies)
    to probabilistically cluster contigs into MAG bins
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:55abb493ec3ad0d976ff7f2ac28eefc7a50887d12b551caa9f1ff8675b9cbeb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - MetaBAT2
  - nf-core/mag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metagenomic-contig-depth-estimation

## Summary

Reconstructs the per-contig sequencing-depth estimation step of the nf-core/mag pipeline: short reads of each sample are mapped to assembled contigs with Bowtie2 and mean per-contig depths are aggregated across samples in the same user-specified group so MetaBAT2 can use co-abundance patterns for MAG binning.

## When to use

Use when you have assembled metagenomic contigs plus per-sample short-read FASTQs and need per-contig mean sequencing depths across samples to feed co-abundance-based binning (e.g., MetaBAT2) or to estimate MAG abundances.

## When NOT to use

- Input is long reads only — nf-core/mag routes long-read alignment to minimap2, not Bowtie2, so the Bowtie2 short-read depth workflow does not apply
- No multi-sample or group information exists and co-abundance-based binning is not the goal — single-sample depth may not support co-abundance clustering
- Contigs have not yet been produced (raw reads only) — assembly must precede depth estimation

## Inputs

- assembled metagenomic contigs (FASTA)
- per-sample short-read sequencing FASTQ files (e.g., CAMISIM simulated reads, Zenodo 10.5281/zenodo.5155395)
- sample-to-group mapping used to define which samples share co-abundance computation

## Outputs

- per-contig mean sequencing depth values for every sample
- group-aggregated per-contig depth/co-abundance table supplied to MetaBAT2
- per-sample MAG abundance table (depth summed over bin contigs, normalized by bin length)

## How to apply

Index the assembled contigs with bowtie2-build, then map each sample's reads to the contigs; nf-core/mag gates alignment by read type, using Bowtie2 for short reads and minimap2 for long reads. Compute the per-contig mean sequencing depth for every sample from the alignments. Aggregate depth and sequence information across all samples in the user-specified group so that co-abundance patterns can be computed, since nf-core/mag utilizes sample-wise group information to compute co-abundances used for the genome binning step. MetaBAT2 then combines tetranucleotide nucleotide frequencies with the co-abundance distance to probabilistically cluster contigs into MAG bins, and per-sample MAG abundance is estimated by summing read depth over a bin's contigs normalized by bin length.

## Related tools

- **nf-core/mag** (Pipeline orchestrating the mapping, per-contig depth computation, group-wise aggregation, and handoff of depths/co-abundances to MetaBAT2 for binning) — https://github.com/nf-core/mag
- **Bowtie2** (Aligns each sample's short reads to the indexed contigs; bowtie2-build indexes contigs before mapping)
- **MetaBAT2** (Consumes the per-contig depths and co-abundances (with nucleotide frequencies) to probabilistically cluster contigs into MAG bins)

## Examples

```
bowtie2-build contigs.fa contigs_idx; bowtie2 -x contigs_idx sample1_R1.fq.gz sample1_R2.fq.gz | samtools sort -o sample1.bam
```

## Evaluation signals

- A complete per-sample abundance table is produced for every sample in the group, consistent with nf-core/mag's automated nf-test assertions on bundled test data
- A non-empty set of MAG bin FASTA files is emitted after MetaBAT2 clustering
- Each sample's reads map successfully to the bowtie2-build-indexed contigs (non-empty sorted BAM per sample)
- Per-sample MAG abundances equal read depth summed over each bin's contigs normalized by bin length (spot-checkable invariant)
- Depths and co-abundances are computed only over samples within the user-specified group

## Limitations

- Alignment strategy is read-type dependent: Bowtie2 for short reads, minimap2 for long reads, so mixing technologies requires separate handling
- Co-abundance computation depends on the user-specified sample grouping; bins reflect co-abundance within the group by default
- Bin quality assessment/refinement (e.g., CheckM2, DAS_Tool) is downstream and out of scope of this depth-estimation step

## Evidence

- [discussion] Group-wise co-abundance feeds binning: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
- [methods] Read-type-dependent alignment: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [abstract] Binning step in the reference workflow: "utilize sample-wise group-information for co-assembly and genome binning"
- [intro] Contigs and bins position in the workflow: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
