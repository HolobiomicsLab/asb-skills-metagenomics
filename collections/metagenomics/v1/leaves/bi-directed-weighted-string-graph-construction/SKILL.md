---
name: bi-directed-weighted-string-graph-construction
description: Use when you already have strain-separated contigs plus long-read alignments
  to them and need to order/connect contigs into scaffolds via a read-supported string
  graph rather than a contig graph.
when_to_use_negative:
- Input contigs are already strain-oblivious and not strain-separated — the scaffolding
  component is downstream of Strainberry's phasing/haplotyping iterations (strainberry_nK
  directories)
- Data is short-read based — the paper notes prior strain-level assembly methods based
  on short reads cannot be re-used with long reads, and this graph relies on long
  dovetail read alignments
- You need to resolve plasmid genomes or more than two strains within a single contig
  — the paper explicitly leaves plasmid resolution and variable strain multiplicity
  within contigs unhandled
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Strainberry
  role: 'Implements the scaffolding step: dovetail alignment filtering, bi-directed
    weighted string graph construction, trimming, path tracing, and scaffold output'
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads to contigs (and re-aligns reads to final scaffolds for assembly.scaffolds.bam)
- name: samtools
  role: Sorts and indexes the read alignment BAM and indexes the FASTA inputs/outputs
    (faidx/index)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:25c6796d75209627bbd75d3e37d3cd0a07a77edc610a0d533077760e2d7f656f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bi-directed-weighted-string-graph-construction

## Summary

Constructs a bi-directed weighted string graph over contig ends from dovetail long-read alignments, the core of Strainberry's strain-aware scaffolding step that joins strain-separated contigs into longer scaffolds.

## When to use

Use when you already have strain-separated contigs plus long-read alignments to them and need to order/connect contigs into scaffolds via a read-supported string graph rather than a contig graph.

## When NOT to use

- Input contigs are already strain-oblivious and not strain-separated — the scaffolding component is downstream of Strainberry's phasing/haplotyping iterations (strainberry_nK directories)
- Data is short-read based — the paper notes prior strain-level assembly methods based on short reads cannot be re-used with long reads, and this graph relies on long dovetail read alignments
- You need to resolve plasmid genomes or more than two strains within a single contig — the paper explicitly leaves plasmid resolution and variable strain multiplicity within contigs unhandled

## Inputs

- Strain-oblivious/strain-separated contigs in FASTA format (e.g. Mock3 ssFlye contigs, assembly.fasta)
- Long reads in FASTQ (e.g. Mock3 PacBio reads.fq.gz)
- Coordinate-sorted, samtools-indexed read-to-contig alignment BAM

## Outputs

- Scaffolds in FASTA format (assembly.scaffolds.fa with .fai index)
- Minimap2-based read alignment on the scaffolds (assembly.scaffolds.bam with .bai index)

## How to apply

Align the input long reads to the strain-separated contigs with minimap2 and sort/index the BAM with samtools. Filter read-to-contig dovetail alignments keeping only high-confidence matches with MAPQ > 40, then build the bi-directed weighted string graph over contig ends, admitting edges according to the paper's four edge-allowance rules. Trim the graph by removing transitive edges and weak edges supported by fewer than 10 reads or less than 90% of a contig end's outgoing read support. Finally trace maximal unambiguous paths through the trimmed graph to define scaffolds, estimating inter-contig gaps as the median distance between reads spanning adjacent contigs, and write the result to assembly.scaffolds.fa.

## Related tools

- **Strainberry** (Implements the scaffolding step: dovetail alignment filtering, bi-directed weighted string graph construction, trimming, path tracing, and scaffold output) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads to contigs (and re-aligns reads to final scaffolds for assembly.scaffolds.bam))
- **samtools** (Sorts and indexes the read alignment BAM and indexes the FASTA inputs/outputs (faidx/index))

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- Scaffold graph retains only edges passing MAPQ > 40 dovetail filters and survives trimming against the <10-read and <90%-of-end-support thresholds
- Traced paths are maximal and unambiguous; every contig used is spanned by reads supporting adjacency
- Inter-contig gap sizes equal the median distance between reads spanning adjacent contigs, not arbitrary fillers
- assembly.scaffolds.fa and a valid indexed assembly.scaffolds.bam (reads re-aligned to scaffolds) are present in the output directory
- On Mock3, benchmark scaffolds against reference strain genomes expecting near-complete reference coverage and ~99.9% base accuracy

## Limitations

- Trimming thresholds (10 reads, 90% support) may drop genuine adjacencies in low-coverage regions
- Scaffolding quality depends on read length; DNA extraction limits N50 to around 5 Kbp in the paper's datasets
- Cannot separate a contig into a variable number of strains when strain variability is high within the same contig
- Graph is built only from dovetail read-to-contig alignments; alignment artifacts or low MAPQ dovetails remove connectivity

## Evidence

- [other] Filter read-to-contig dovetail alignments, keeping only high-confidence matches with MAPQ > 40.: [evidence span withheld — anchor status: not_found]
- [other] Build the bi-directed weighted string graph over contig ends, admitting edges according to the paper's four edge-allowance rules.: [evidence span withheld — anchor status: not_found]
- [discussion] Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component: "Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component"
- [readme] A minimap2-based alignment of input reads on the output assembly is also available in the assembly.scaffolds.bam file.: "A minimap2-based alignment of input reads on the output assembly is also available in the `assembly.scaffolds.bam` file."
- [other] Combines a strain-oblivious assembler with a scaffolding component: align reads to contigs and join contigs into longer scaffolds.: [evidence span withheld — anchor status: not_found]
