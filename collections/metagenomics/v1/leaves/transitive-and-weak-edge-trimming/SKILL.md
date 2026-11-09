---
name: transitive-and-weak-edge-trimming
description: Use when strain-separated metagenome contigs must be joined into scaffolds
  using long-read dovetail alignments, requiring removal of transitive and weakly
  supported edges from a string graph before path tracing.
when_to_use_negative:
- Input is a short-read metagenome assembly or short-read alignment — Strainberry
  relies uniquely on long-read data and prior strain-level assembly methods based
  on short reads cannot be reused
- Strain-separated contigs are not available yet — this trimming operates on the scaffolding
  graph of already strain-separated contigs, not on the initial strain-oblivious assembly
- Plasmid genome resolution is the goal — the paper did not evaluate Strainberry's
  ability to resolve plasmid genomes
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: 'Implements the strain-aware scaffolding component: dovetail alignment filtering,
    bi-directed string graph construction, transitive/weak edge trimming, and scaffold
    path tracing'
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads to the strain-oblivious contigs and produces the final read
    alignment on the output scaffolds (assembly.scaffolds.bam)
- name: samtools
  role: Sorts and indexes the read-to-contig BAM alignment and indexes FASTA files
    (faidx) required by Strainberry
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:d32dcf7ae6ac74ff7c793ff2937114389d98ab59cfe6ac76b0e2d6976c70b9fe
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Transitive and weak edge trimming

## Summary

Graph-pruning step of Strainberry's strain-aware scaffolding component: dovetail read-to-contig alignments are used to build a bi-directed weighted string graph over contig ends, from which transitive edges and weak edges (low read support or low relative support) are removed before tracing scaffold paths.

## When to use

Use when strain-separated metagenome contigs must be joined into scaffolds using long-read dovetail alignments, requiring removal of transitive and weakly supported edges from a string graph before path tracing.

## When NOT to use

- Input is a short-read metagenome assembly or short-read alignment — Strainberry relies uniquely on long-read data and prior strain-level assembly methods based on short reads cannot be reused
- Strain-separated contigs are not available yet — this trimming operates on the scaffolding graph of already strain-separated contigs, not on the initial strain-oblivious assembly
- Plasmid genome resolution is the goal — the paper did not evaluate Strainberry's ability to resolve plasmid genomes

## Inputs

- Strain-oblivious metagenome assembly in FASTA format (e.g., Mock3 ssFlye contigs)
- Coordinate-sorted long-read alignment in BAM format (indexed with samtools index; assembly FASTA indexed with samtools faidx)
- Raw long reads in FASTQ (used with minimap2 to produce the alignment)

## Outputs

- Final strain-resolved scaffolds in assembly.scaffolds.fa (with .fai index)
- minimap2-based read alignment on the scaffolds: assembly.scaffolds.bam (with .bai index)

## How to apply

Align the input long reads (e.g., Mock3 reads from task_001) to the strain-oblivious contigs (e.g., ssFlye Mock3 assembly in FASTA) with minimap2 and sort/index the BAM with samtools. Filter read-to-contig dovetail alignments to high-confidence matches with MAPQ > 40, then build the bi-directed weighted string graph over contig ends admitting edges according to the paper's four edge-allowance rules. Trim the graph by removing transitive edges and weak edges supported by fewer than 10 reads or by less than 90% of a contig end's outgoing read support; this rationale follows the paper's use of 'well-established tools for haplotype phasing and genome assembly'. After trimming, trace maximal unambiguous paths to define scaffolds, estimating inter-contig gaps as the median distance between reads spanning adjacent contigs, and write assembly.scaffolds.fa plus a minimap2-based read alignment on the scaffolds (assembly.scaffolds.bam).

## Related tools

- **Strainberry** (Implements the strain-aware scaffolding component: dovetail alignment filtering, bi-directed string graph construction, transitive/weak edge trimming, and scaffold path tracing) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads to the strain-oblivious contigs and produces the final read alignment on the output scaffolds (assembly.scaffolds.bam))
- **samtools** (Sorts and indexes the read-to-contig BAM alignment and indexes FASTA files (faidx) required by Strainberry)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- Final assembly.scaffolds.fa contains joined scaffolds (contigs merged), and assembly.scaffolds.bam exists with a valid .bai index
- All dovetail alignments admitted to the graph have MAPQ > 40
- Retained edges are supported by at least 10 reads and at least 90% of the corresponding contig end's outgoing read support
- Benchmark Mock3 scaffolds against reference strain genomes: near-complete reference coverage and approximately 99.9% base accuracy, as reported for Strainberry on mock communities
- Inter-contig gaps correspond to the median distance between reads spanning adjacent contigs

## Limitations

- Haplotype phasing assumption of evenly split read coverage does not hold in metagenomic settings where strains exhibit different abundance/coverage levels
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Limited read length (around N50 of 5 Kbp) due to lack of easy high-molecular-weight DNA extraction methods for all bacteria can constrain scaffolding
- Relationship between depth of coverage (60–80× suffices for near-complete strain coverage), read length, and haplotype resolution ability merits further study

## Evidence

- [discussion] Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component: "Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component"
- [intro] Combines strain-oblivious assembler with scaffolding: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component"
- [readme] Output files of Strainberry run: "Strainberry output assembly is stored in the `assembly.scaffolds.fa` file.
A minimap2-based alignment of input reads on the output assembly is also available"
- [readme] Input format requirements for Strainberry: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [abstract] Benchmark accuracy on mock communities: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
