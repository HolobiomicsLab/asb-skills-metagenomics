---
name: metagenome-overlap-graph-assembly
description: Use when assembling individual strain-level genomes de novo from Illumina-type
  metagenomic short reads without a reference, especially for high-complexity communities
  with many closely related strains.
when_to_use_negative:
- Third-generation (long, high-error-rate) sequencing reads — the article states 'The
  great Error Rate of third generation sequencing reads, however, introduce a range
  of issues that still need to be resolved'
- When maximum contig length/N50 rather than strain resolution is the priority — MEGAHIT
  'outperforms all other methods in terms of contig length (N50 in particular)'
- When a reference genome is available and reference-assisted assembly is preferred
  — StrainXpress is explicitly de novo and 'free of biases' but reference-assisted
  assembly may be more appropriate
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: 'Main pipeline: clusters reads, performs local assembly per cluster, and builds
    the master overlap graph to produce master contigs'
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes pairwise overlaps between haplotigs used to build the master overlap
    graph in global assembly
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress; its SAVAGE overlap-graph lineage inspired the
    overlap thresholds (overlap >100 bp, identity >=0.99)
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:8984e9e1b19d657267c172a6de17c5c318a253179fc458bf9fa79279bdd68a91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metagenome-overlap-graph-assembly

## Summary

StrainXpress's three-stage de novo strain-aware metagenome assembly (Clustering Reads, Local Assembly, Global Assembly) that combines cluster-specific haplotigs into final master contigs via a master overlap graph, overcoming overlap-graph computational bottlenecks for metagenomes with up to >1000 strains.

## When to use

Use when assembling individual strain-level genomes de novo from Illumina-type metagenomic short reads without a reference, especially for high-complexity communities with many closely related strains.

## When NOT to use

- Third-generation (long, high-error-rate) sequencing reads — the article states 'The great Error Rate of third generation sequencing reads, however, introduce a range of issues that still need to be resolved'
- When maximum contig length/N50 rather than strain resolution is the priority — MEGAHIT 'outperforms all other methods in terms of contig length (N50 in particular)'
- When a reference genome is available and reference-assisted assembly is preferred — StrainXpress is explicitly de novo and 'free of biases' but reference-assisted assembly may be more appropriate

## Inputs

- Interleaved FASTQ file of Illumina metagenomic short reads (e.g. all_reads.fq)

## Outputs

- final_contigs.fasta in the stageb folder: master contigs assembled at strain resolution
- stage-a output: cluster-specific haplotigs from local assembly of each read cluster

## How to apply

Run StrainXpress on interleaved FASTQ short reads: first cluster reads into strain/cluster groups, then perform local assembly of each cluster to produce haplotigs (stage-a output). For global assembly, build a master overlap graph over cluster-specific haplotigs by computing pairwise overlaps with minimap2, retaining edges with overlap length >100 bp and identity >=0.99 (thresholds inspired by earlier overlap-graph work such as SAVAGE). Simplify the graph by removing transitive edges and resolving branch (bifurcation) structures, then iteratively extend contigs only along unique unambiguous paths until no contig can be extended further. Write master contigs as FASTA to the stageb folder as final_contigs.fasta; for large datasets use the -fast cluster method.

## Related tools

- **StrainXpress** (Main pipeline: clusters reads, performs local assembly per cluster, and builds the master overlap graph to produce master contigs) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes pairwise overlaps between haplotigs used to build the master overlap graph in global assembly) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress; its SAVAGE overlap-graph lineage inspired the overlap thresholds (overlap >100 bp, identity >=0.99)) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- final_contigs.fasta exists in the stageb output folder and is a well-formed FASTA file of assembled contig sequences
- Genome Fraction of assembled contigs should approach the reported benchmark (93.45% on low complexity 2x250 bp data vs 74.74% for SPAdes)
- Assembly identity should be very high (StrainXpress reported 99.93% vs IDBA-UD's 99.91%)
- Strain-specific reconstructed sequence amount should exceed competing assemblers (average +26.75% across all data sets)
- Graph simplification completeness: iterative extension terminates when no contig can be extended further along unique paths

## Limitations

- The short length of Illumina-type reads constrains contig length, preventing computation of strain-specific contigs that span entire genomes
- High error rate of third-generation sequencing reads introduces unresolved issues
- Poorly covered strains may be challenging; StrainXpress claims to handle strains down to ~10x average coverage (high complexity data set at 10x per strain)

## Evidence

- [results] Three-stage overview: [evidence span withheld — anchor status: not_found]
- [readme] OLC paradigm description: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data at strain resolution."
- [abstract] Abstract capability claim: "StrainXpress reconstructs strain-specific genomes from metagenomes that involve up to >1000 strains and proves to successfully deal with poorly covered strains."
- [results] Performance vs SPAdes: "StrainXpress: 93.45% versus 74.74% by SPAdes, as the second highest scoring"
- [readme] Usage invocation: "python ../scripts/strainxpress.py -fq all_reads.fq"
