---
name: nucleotide-frequency-composition-analysis
description: Use when assembled contigs and multi-sample short-read data are available
  and contigs must be clustered into MAGs using sequence composition plus per-sample
  depth/co-abundance signals.
when_to_use_negative:
- Only a single sample is available and no group of samples exists, so group-wise
  co-abundance cannot inform binning
- The contigs are already binned or the input is a MAG FASTA set rather than raw contigs
- Bin refinement/refinement of existing bins is the goal — nf-core/mag 2.1.0 does
  not include a bin refinement step
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaBAT2
  role: 'Binning tool: combines tetranucleotide nucleotide frequencies with co-abundance
    distances to probabilistically cluster contigs into MAG bins'
- name: Bowtie2
  role: Maps short reads of each sample to contigs to compute per-contig mean sequencing
    depth for co-abundance estimation
- name: nf-core/mag
  role: Nextflow (DSL2) pipeline that orchestrates mapping, depth aggregation by sample
    group, MetaBAT2 binning, and MAG abundance estimation
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:a1ec5dd32f298fffe7254e5bbfdda6165ec99df8d1b5575dd624e36397537ba9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - MetaBAT2
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# nucleotide-frequency-composition-analysis

## Summary

Binning of assembled metagenomic contigs into MAGs using nucleotide (tetranucleotide) frequencies combined with group-wise co-abundance patterns, as implemented for MetaBAT2 in the nf-core/mag pipeline.

## When to use

Use when assembled contigs and multi-sample short-read data are available and contigs must be clustered into MAGs using sequence composition plus per-sample depth/co-abundance signals.

## When NOT to use

- Only a single sample is available and no group of samples exists, so group-wise co-abundance cannot inform binning
- The contigs are already binned or the input is a MAG FASTA set rather than raw contigs
- Bin refinement/refinement of existing bins is the goal — nf-core/mag 2.1.0 does not include a bin refinement step

## Inputs

- Assembled contigs FASTA (e.g., from nf-core/mag assembly step)
- Per-sample short-read FASTQ files (e.g., CAMISIM simulated metagenomic reads, Zenodo 10.5281/zenodo.5155395)
- Bowtie2 index of the contigs (bowtie2-build)
- Sample-to-group specification for co-abundance aggregation

## Outputs

- MAG bin FASTA files (one per metagenome assembled genome)
- Per-contig sequencing depth table across samples
- Per-sample MAG abundance table (depth summed over bin contigs, normalized by bin length)

## How to apply

Index the assembled contigs with bowtie2-build, then map each sample's short reads with Bowtie2 (nf-core/mag gates alignment by read type: Bowtie2 for short reads, minimap2 for long reads) and compute per-contig mean sequencing depth per sample. Aggregate depth values across all samples in the user-specified group so co-abundance patterns can be calculated. Run MetaBAT2 on the contigs supplying the per-contig depth table; MetaBAT2 combines tetranucleotide nucleotide frequencies with the co-abundance distance to probabilistically cluster contigs into MAG bins. Finally, estimate per-sample MAG abundance by summing read depth over the contigs of each bin, normalized by bin length. Grouping by sample group by default reflects the pipeline's design to 'utilize sample-wise group-information for co-assembly and genome binning'.

## Related tools

- **MetaBAT2** (Binning tool: combines tetranucleotide nucleotide frequencies with co-abundance distances to probabilistically cluster contigs into MAG bins)
- **Bowtie2** (Maps short reads of each sample to contigs to compute per-contig mean sequencing depth for co-abundance estimation)
- **nf-core/mag** (Nextflow (DSL2) pipeline that orchestrates mapping, depth aggregation by sample group, MetaBAT2 binning, and MAG abundance estimation) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag --input samplesheet.csv --outdir results -profile docker
```

## Evaluation signals

- A non-empty set of MAG bin FASTA files is produced for every sample group
- A complete per-sample MAG abundance table is emitted, with values consistent with depth-sum-over-bin-contigs normalized by bin length
- Per-contig mean sequencing depth is computed for every sample before binning (not just the sample that supplied the contigs)
- Pipeline's automated nf-test assertions pass on the bundled CAMISIM test data (Zenodo 10.5281/zenodo.5155395)
- Binning used both nucleotide frequencies and co-abundance information within the same sample group, matching MetaBAT2's probabilistic clustering inputs

## Limitations

- Co-abundance is computed within the user-specified sample group by default; samples outside the group do not contribute to binning signals
- Alignment is gated by read type: Bowtie2 only for short reads, minimap2 for long reads — using the wrong aligner for a read type is incorrect in this workflow
- nf-core/mag 2.1.0 does not yet include functional annotation, assembly refinement, or bin refinement steps, so bins are not refined after MetaBAT2

## Evidence

- [discussion] Group-wise co-abundance for binning: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
- [abstract] Group information for binning: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [readme] Composition + multi-sample coverage binning: "A program for unsupervised binning of metagenomic contigs by using nucleotide composition, coverage data in multiple samples and linkage data from paired end reads."
- [intro] Binning in MAG workflow: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
