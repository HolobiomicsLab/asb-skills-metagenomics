---
name: fasta-contig-output-generation
description: Use when the goal is to combine cluster-specific contigs (haplotigs)
  from a strain-aware metagenomic assembly into final master contigs using an overlap
  graph and emit them as a FASTA output file.
when_to_use_negative:
- Input consists of high-error third-generation sequencing reads (PacBio/ONT) — the
  paper notes the great error rate of such reads 'introduce a range of issues that
  still need to be resolved'; StrainXpress targets Illumina-type short reads.
- Only a species-level (not strain-resolved) metagenome assembly is needed — general
  assemblers like SPAdes, MEGAHIT or IDBA-UD are the stated alternatives, and MEGAHIT
  outperforms all methods on contig length (N50).
- Input data is already a FASTA of contigs/feature output rather than sequencing reads
  — the pipeline consumes interleaved FASTQ reads as its entry point.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0102
tools:
- name: StrainXpress
  role: De novo OLC-based assembler whose Clustering Reads, Local Assembly and Global
    Assembly stages produce the final master contigs emitted as FASTA
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes pairwise overlaps between cluster-specific haplotigs used to build
    the master overlap graph
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Declared dependency of the StrainXpress installation underlying the local-assembly
    component
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:e0bf41990edfa80220294075ef49ce63937ec85c83b0ec93379aec9b5854411a
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fasta-contig-output-generation

## Summary

StrainXpress global assembly stage: cluster-specific haplotigs from local assembly are merged into strain-resolved master contigs via a minimap2-built overlap graph, and the result is written as FASTA (final_contigs.fasta) to the stageb folder.

## When to use

Use when the goal is to combine cluster-specific contigs (haplotigs) from a strain-aware metagenomic assembly into final master contigs using an overlap graph and emit them as a FASTA output file.

## When NOT to use

- Input consists of high-error third-generation sequencing reads (PacBio/ONT) — the paper notes the great error rate of such reads 'introduce a range of issues that still need to be resolved'; StrainXpress targets Illumina-type short reads.
- Only a species-level (not strain-resolved) metagenome assembly is needed — general assemblers like SPAdes, MEGAHIT or IDBA-UD are the stated alternatives, and MEGAHIT outperforms all methods on contig length (N50).
- Input data is already a FASTA of contigs/feature output rather than sequencing reads — the pipeline consumes interleaved FASTQ reads as its entry point.

## Inputs

- Interleaved FASTQ file of metagenomic Illumina short reads (e.g. all_reads.fq)
- Haplotigs (cluster-specific contigs) produced by StrainXpress local assembly of each read cluster (stage-a output)

## Outputs

- final_contigs.fasta — well-formed FASTA file of assembled strain-resolved master contigs, written to the stageb output folder

## How to apply

Run StrainXpress so that its two-stage pipeline first produces haplotigs by local assembly of each read cluster (stage-a output), then collect these haplotigs as the node sequences for global assembly. Compute pairwise overlaps between haplotigs with minimap2 and build a master overlap graph, retaining edges only when the overlap length exceeds 100 bp at an identity of at least 0.99 — thresholds inspired by earlier overlap-graph work. Simplify the graph by removing transitive edges and resolving branch (bifurcation) structures, then iteratively extend contigs only along unique, unambiguous paths, repeating until no contig can be extended further. Write the resulting master contigs as FASTA to the stageb folder as final_contigs.fasta; the whole pipeline is invoked via the strainxpress.py script on interleaved FASTQ input (optionally with -fast for large data sets).

## Related tools

- **StrainXpress** (De novo OLC-based assembler whose Clustering Reads, Local Assembly and Global Assembly stages produce the final master contigs emitted as FASTA) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes pairwise overlaps between cluster-specific haplotigs used to build the master overlap graph) — https://github.com/lh3/minimap2
- **HaploConduct** (Declared dependency of the StrainXpress installation underlying the local-assembly component) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- final_contigs.fasta exists in the stageb output folder and parses as a well-formed FASTA file of assembled contig sequences.
- Master overlap graph edges are restricted to minimap2 overlaps of length >100 bp at identity >=0.99.
- Graph simplification removed transitive edges and resolved bifurcation structures before any contig extension.
- The iterative extension has reached a fixpoint: no remaining contig can be extended along a unique path.
- Assembled contig quality can be cross-checked with Quast/metaQuast; on low-complexity 2x250 bp data StrainXpress reached Genome Fraction 93.45% (vs 74.74% for SPAdes).

## Limitations

- The short length of Illumina-type reads puts constraints on contig length, preventing computation of strain-specific contigs that span whole genomes.
- The high error rate of third-generation sequencing reads introduces unresolved issues; the method is built for linux-based systems and python3 only.
- On large data sets the -fast clustering mode is recommended, and clustering/local-assembly speed-ups (46x and 8x respectively) matter for high-complexity inputs such as 1057 strains from 376 species.

## Evidence

- [readme] StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data at strain resolution.: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm"
- [results] Pipeline stages are named explicitly in the article.: "the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
- [discussion] Overlap graphs are the key substrate for strain resolution.: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
- [readme] Output location stated in the README.: "The result is in the stageb folder"
- [intro] Motivation for a comprehensive overlap-graph-based solution.: "overcoming the computational bottlenecks of OG-based approaches is key to success when aiming at considerably improved haplotype/strain aware assemblies of metagenomes"
