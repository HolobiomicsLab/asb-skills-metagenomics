---
name: kmer-coverage-depth-estimation
description: Use when reproducing MetaPlatanus' coverage-depth-based exclusion of
  interspecies misjoinings or its contig edge-correction, e.g. when contig links or
  terminal regions must be judged against per-contig k-mer coverage depth.
when_to_use_negative:
- Input lacks paired short reads — MetaPlatanus requires at least one short-read paired-end
  library, so coverage depth cannot be estimated reliably
- Goal is polishing/gap closing (e.g. TGS-GapCloser, NextPolish) rather than coverage-based
  misjoin exclusion or edge trimming
- Sequences are too short for reliable k-mer statistics — the 4-mer distribution is
  set to 1 for contigs shorter than 1000 bp, so composition-based filtering is disabled
  there
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: MetaPlatanus
  role: Implements the coverage-depth estimation and combines it with 4-mer frequencies
    and long-range links to exclude interspecies misjoins and trim contig edges
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembles the initial contigs on which k-mer coverage depths are computed
    (contig-assembly k-mer coverage cutoff default 4 with MEGAHIT)
  repo: https://github.com/voutcn/megahit
- name: minimap2
  role: Maps short-read pairs onto contigs to derive link support for the scaffold
    graph
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:3b4ecacbed4eb5f4ddc27e6a3215c5dd8693929b1bdc615a909401e164bd1b8c
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# kmer-coverage-depth-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates per-contig k-mer coverage depth from short reads (Illumina paired-end FASTQ) and uses the coverage-depth signal, together with 4-mer frequency compositions, to exclude interspecies misjoinings during MetaPlatanus scaffolding and to correct contig edges whose coverage deviates from the contig median.

## When to use

Use when reproducing MetaPlatanus' coverage-depth-based exclusion of interspecies misjoinings or its contig edge-correction, e.g. when contig links or terminal regions must be judged against per-contig k-mer coverage depth.

## When NOT to use

- Input lacks paired short reads — MetaPlatanus requires at least one short-read paired-end library, so coverage depth cannot be estimated reliably
- Goal is polishing/gap closing (e.g. TGS-GapCloser, NextPolish) rather than coverage-based misjoin exclusion or edge trimming
- Sequences are too short for reliable k-mer statistics — the 4-mer distribution is set to 1 for contigs shorter than 1000 bp, so composition-based filtering is disabled there

## Inputs

- Contig sequences in FASTA format (e.g. MEGAHIT initial contigs)
- Short-read paired-end libraries in FASTQ (Illumina; at least one required by MetaPlatanus)
- Read-to-contig alignments (e.g. SAM/PAF from BWA or minimap2) to link contig pairs

## Outputs

- Per-contig k-mer coverage depth table
- misjoin_exclusion_results.tsv with retained/removed links and their posterior probabilities
- trimmed_contigs.fasta and edge_trim_report.tsv (original length, trimmed length, median coverage, trimmed-edge coordinates)

## How to apply

Count k-mers from the Illumina paired-end short reads to compute a per-contig coverage-depth table for the contig FASTA (initial contigs typically produced by MEGAHIT, which MetaPlatanus combines for contig-assembly with a k-mer coverage cutoff default of 4 with MEGAHIT, 2 otherwise). For scaffolding, compute for each contig pair with at least two paired-read links the coverage depth ratio min(c1,c2)/max(c1,c2) and build the empirical complementary cumulative distribution F_coverage; exclude a link when F_coverage(r) x F_4-mer(d) < 0.05, setting F_4-mer to 1 for contigs shorter than 1000 bp since they are too short for reliable 4-mer frequencies. For contig edge correction, scan from both contig ends and mark edge regions where local k-mer coverage violates the 1.5x deviation condition relative to the contig median coverage, then split and discard the offending edge segment. Rationale: species-specific coverage depth and sequence composition are the features MetaPlatanus iteratively uses to prevent interspecies misjoinings while scaffolding and to correct mis-assemblies.

## Related tools

- **MetaPlatanus** (Implements the coverage-depth estimation and combines it with 4-mer frequencies and long-range links to exclude interspecies misjoins and trim contig edges) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembles the initial contigs on which k-mer coverage depths are computed (contig-assembly k-mer coverage cutoff default 4 with MEGAHIT)) — https://github.com/voutcn/megahit
- **minimap2** (Maps short-read pairs onto contigs to derive link support for the scaffold graph) — https://github.com/lh3/minimap2

## Evaluation signals

- Links planted between simulated cross-species contigs (differing coverage depth and oligonucleotide composition) are excluded while within-species links are retained in misjoin_exclusion_results.tsv
- Every link considered in the exclusion test is supported by at least two paired reads in the scaffold graph
- F_4-mer equals 1 for any contig shorter than 1000 bp, so those pairs are judged solely by F_coverage
- Trimmed-edge coordinates in edge_trim_report.tsv fall within the expected region of the known synthetic coverage profile
- Every retained region after edge trimming satisfies the 1.5x condition relative to the contig median coverage

## Limitations

- Contigs shorter than 1000 bp cannot provide reliable 4-mer frequency estimates, weakening composition-based discrimination
- The exclusion rule depends on empirical complementary cumulative distributions estimated from the dataset, so very small or low-diversity mock communities may yield poorly calibrated F_coverage / F_4-mer
- Coverage-depth discrimination degrades for species with similar coverage depths, which is precisely why an assembler able to join sequences using a small number of links while avoiding misjoins is needed for low-abundance species

## Evidence
