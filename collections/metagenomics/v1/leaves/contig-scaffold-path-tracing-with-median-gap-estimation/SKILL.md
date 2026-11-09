---
name: contig-scaffold-path-tracing-with-median-gap-estimation
description: Use when a strain-oblivious long-read metagenome assembly (FASTA) and
  a coordinate-sorted long-read alignment (BAM) exist and strain-separated contigs
  must be joined into longer, more complete strain scaffolds.
when_to_use_negative:
- The input is a single-isolate genome or a high-complexity metagenome rather than
  a single-sample low-complexity metagenome, which is Strainberry's stated scope
- No long reads or read-to-contig alignment are available — the scaffolding graph
  depends entirely on dovetail read alignments spanning contig ends
- The contigs already come as finished scaffolds and no further joining is needed
  — this step adds no information over the existing read alignment
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Strainberry
  role: 'Implements the strain-aware scaffolding: builds the bi-directed weighted
    string graph over contig ends, trims edges, traces scaffold paths, and writes
    assembly.scaffolds.fa'
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads to the strain-separated contigs (and produces the final
    scaffolds read alignment in assembly.scaffolds.bam) to supply dovetail alignments
    for graph edges
- name: samtools
  role: Sorts and indexes the read alignment BAM and faidx-indexes the contig FASTA,
    the input formats Strainberry requires
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:7b76f51bb85a1341d09699d5496139fb5468e9ad629dacdfae8ac612d3d32cba
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Contig scaffold path tracing with median gap estimation

## Summary

A strain-aware scaffolding method from Strainberry that aligns long reads to strain-separated contigs, builds a bi-directed weighted string graph over contig ends using dovetail read alignments, trims weak/transitive edges, and traces maximal unambiguous paths to join strain-separated contigs into longer scaffolds with median-estimated gaps.

## When to use

Use when a strain-oblivious long-read metagenome assembly (FASTA) and a coordinate-sorted long-read alignment (BAM) exist and strain-separated contigs must be joined into longer, more complete strain scaffolds.

## When NOT to use

- The input is a single-isolate genome or a high-complexity metagenome rather than a single-sample low-complexity metagenome, which is Strainberry's stated scope
- No long reads or read-to-contig alignment are available — the scaffolding graph depends entirely on dovetail read alignments spanning contig ends
- The contigs already come as finished scaffolds and no further joining is needed — this step adds no information over the existing read alignment

## Inputs

- Strain-oblivious metagenome assembly in FASTA format (e.g., Mock3 metaFlye/ssFlye contigs)
- Coordinate-sorted long-read alignment to the contigs in BAM format (minimap2 | samtools sort), indexed with samtools index
- Indexed assembly FASTA (samtools faidx)

## Outputs

- Final strain scaffolds in assembly.scaffolds.fa (with .fai index)
- Minimap2-based read alignment on the output scaffolds: assembly.scaffolds.bam (with .bai index)

## How to apply

Align the input long reads to the strain-oblivious/strain-separated contigs with minimap2 and sort/index the result with samtools, since Strainberry expects a coordinate-sorted, indexed BAM (samtools index) alongside an faidx-indexed FASTA. Filter read-to-contig dovetail alignments to high-confidence matches (MAPQ > 40) and build the bi-directed weighted string graph over contig ends, admitting edges according to the paper's four edge-allowance rules. Trim the graph by removing transitive edges and weak edges supported by fewer than 10 reads or by less than 90% of a contig end's outgoing read support. Then trace maximal unambiguous paths through the trimmed graph to define scaffolds, estimating inter-contig gaps as the median distance between reads spanning adjacent contigs, and write the final scaffolds to assembly.scaffolds.fa.

## Related tools

- **Strainberry** (Implements the strain-aware scaffolding: builds the bi-directed weighted string graph over contig ends, trims edges, traces scaffold paths, and writes assembly.scaffolds.fa) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads to the strain-separated contigs (and produces the final scaffolds read alignment in assembly.scaffolds.bam) to supply dovetail alignments for graph edges)
- **samtools** (Sorts and indexes the read alignment BAM and faidx-indexes the contig FASTA, the input formats Strainberry requires)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- assembly.scaffolds.fa exists in the output directory with a matching .fai index, and assembly.scaffolds.bam/.bai contain a minimap2 read alignment on the scaffolds
- On the Mock3 benchmark, scaffolds should show near-complete reference coverage of the strain genomes with approximately 99.9% base accuracy
- Edges retained in the string graph are supported by at least 10 reads and at least 90% of a contig end's outgoing read support, with transitive edges removed
- Only dovetail read-to-contig alignments with MAPQ > 40 contribute edges
- On the README example, scaffolding should produce two scaffolds (one closer to E. coli K12, one closer to strain W)

## Limitations

- Strainberry was not evaluated for plasmid genome resolution, left to future work by the authors
- With high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Read length is limited (around N50 of 5 Kbp) by lack of easy high-molecular-weight DNA extraction for all bacteria, potentially limiting contig-joining
- The scaffolding outcome is only as good as the underlying haplotype phasing; coverage is not evenly split among strains in metagenomic settings

## Evidence

- [intro] Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding.: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [readme] A minimap2-based alignment of input reads on the output assembly is also available in the assembly.scaffolds.bam file.: "A minimap2-based alignment of input reads on the output assembly is also available in the `assembly.scaffolds.bam` file."
- [abstract] We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy.: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [discussion] Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component: "Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component"
- [readme] Both FASTA and BAM files are expected to be indexed with the samtools faidx and samtools index commands, respectively.: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
