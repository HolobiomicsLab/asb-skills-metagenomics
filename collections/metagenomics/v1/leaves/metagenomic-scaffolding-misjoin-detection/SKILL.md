---
name: metagenomic-scaffolding-misjoin-detection
description: Use when scaffolding metagenomic contigs with paired long-range links
  and you must decide which contig-pair links are probable interspecies misjoins based
  on coverage depth and oligonucleotide composition.
when_to_use_negative:
- Input is a finished or reference-based assembly where binning/coverage-based link
  filtering is unnecessary
- Only single-end short reads are available — MetaPlatanus requires at least one short-read
  paired-end library
- Contigs are all shorter than 1000 bp, so the F_4-mer component is forced to 1 and
  the filter degenerates to coverage ratio alone
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: MetaPlatanus
  role: Implements the scaffold-graph construction and the F_coverage × F_4-mer <
    0.05 link-exclusion criterion during its scaffolding step
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembles the initial contigs from short reads that become the nodes of the
    scaffold graph
  repo: https://github.com/voutcn/megahit
- name: minimap2
  role: Maps short-read pairs onto contigs to establish the paired-read links supporting
    scaffold-graph edges
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:89cdceb545c4723c73cf2dfaeca43ea6e1b66b546778f41b2f257d78b00f1a6c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - MEGAHIT
  - MetaPlatanus
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-scaffolding-misjoin-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs MetaPlatanus's interspecies link-exclusion filter used during scaffolding: links between contigs are tested against empirical complementary cumulative distributions of coverage-depth ratios (F_coverage) and 4-mer frequency distances (F_4-mer), and a link is excluded when F_coverage(r) × F_4-mer(d) < 0.05.

## When to use

Use when scaffolding metagenomic contigs with paired long-range links and you must decide which contig-pair links are probable interspecies misjoins based on coverage depth and oligonucleotide composition.

## When NOT to use

- Input is a finished or reference-based assembly where binning/coverage-based link filtering is unnecessary
- Only single-end short reads are available — MetaPlatanus requires at least one short-read paired-end library
- Contigs are all shorter than 1000 bp, so the F_4-mer component is forced to 1 and the filter degenerates to coverage ratio alone

## Inputs

- Contig FASTA assembled from short reads (e.g. by MEGAHIT)
- Illumina paired-end FASTQ short-read library (required by MetaPlatanus)
- Optional long-range library: mate-pair FASTQ, ONT/PacBio long reads, or 10x linked reads providing linking read pairs

## Outputs

- Scaffold graph with retained within-species links
- List of excluded interspecies links with their posterior probabilities (misjoin_exclusion_results.tsv)

## How to apply

Build a scaffold graph whose edges join contigs supported by at least two paired reads. For each linked pair, compute the coverage depth ratio min(c1,c2)/max(c1,c2) and the Euclidean distance between the contigs' 4-mer frequency vectors, using k-mer counts from short reads (Illumina paired-end FASTQ mapped with BWA or minimap2). Estimate the empirical complementary cumulative distributions F_coverage and F_4-mer from all linked pairs, setting F_4-mer to 1 for contigs shorter than 1000 bp because 4-mer frequencies are unreliable on short sequences. Exclude a link when the posterior-style product F_coverage(r) × F_4-mer(d) < 0.05, and prune it from the scaffold graph so only within-species links drive scaffolding. This exploits the species-specific coverage depth and sequence composition features that MetaPlatanus uses to prevent interspecies misjoinings while scaffolding.

## Related tools

- **MetaPlatanus** (Implements the scaffold-graph construction and the F_coverage × F_4-mer < 0.05 link-exclusion criterion during its scaffolding step) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembles the initial contigs from short reads that become the nodes of the scaffold graph) — https://github.com/voutcn/megahit
- **minimap2** (Maps short-read pairs onto contigs to establish the paired-read links supporting scaffold-graph edges) — https://github.com/lh3/minimap2

## Examples

```
metaplatanus pe mp -m mate.fa -o out; minimap2 -ax sr contigs.fa short_R1.fq short_R2.fq > aln.sam
```

## Evaluation signals

- Links planted between contigs from distinct simulated species (differing coverage depth and oligonucleotide composition) are excluded, while within-species links are retained
- Every excluded link has F_coverage(r) × F_4-mer(d) < 0.05; retained links have product ≥ 0.05
- No link in the scaffold graph is supported by fewer than two paired reads
- Contigs shorter than 1000 bp contribute F_4-mer = 1 in all posterior computations
- Module runs end-to-end on synthetic multi-species FASTA/FASTQ input and writes misjoin_exclusion_results.tsv with per-link posterior probabilities

## Limitations

- F_4-mer is set to 1 for contigs < 1000 bp because 4-mer frequencies are unreliable on very short sequences, weakening the filter to coverage-ratio evidence alone
- Very low-abundance species have few reads, so few links exist; the filter must avoid misjoins using a small number of links
- The filter requires coverage depth and composition to differ between species — closely related species with similar coverage and 4-mer profiles may evade exclusion

## Evidence
