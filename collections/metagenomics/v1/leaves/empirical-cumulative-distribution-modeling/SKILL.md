---
name: empirical-cumulative-distribution-modeling
description: Use when scaffolding metagenomic contigs with paired long-range links
  and you need to detect links joining contigs from different species using coverage
  depth and k-mer composition evidence.
when_to_use_negative:
- Input is a single-isolate genome assembly where coverage/composition divergence
  between species is not a concern
- Contigs are shorter than 1000 bp and the task depends on reliable 4-mer frequency
  distances (F_4-mer is set to 1 there, so it carries no signal)
- Linking evidence is a single read pair per contig pair, since the scaffold graph
  requires at least two paired reads per link
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaPlatanus
  role: Implements the scaffold graph construction and the F_coverage × F_4-mer <
    0.05 interspecies link-exclusion filter during scaffolding
  repo: https://github.com/rkajitani/MetaPlatanus
- name: minimap2
  role: Maps short-read pairs (preset -ax sr) onto contigs to quantify linkage supporting
    each contig pair
  repo: https://github.com/lh3/minimap2
- name: MEGAHIT
  role: Assembles the initial metagenomic contigs whose links are subsequently filtered
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:e459b94a68370e77a5e425ba30ab39bdd059e75f2a67f2b44e1c78c65c27dbca
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# empirical-cumulative-distribution-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds empirical complementary cumulative distributions of contig-pair coverage depth ratios and 4-mer frequency distances, then combines them (F_coverage × F_4-mer < 0.05) to exclude probable interspecies misjoins during MetaPlatanus scaffolding.

## When to use

Use when scaffolding metagenomic contigs with paired long-range links and you need to detect links joining contigs from different species using coverage depth and k-mer composition evidence.

## When NOT to use

- Input is a single-isolate genome assembly where coverage/composition divergence between species is not a concern
- Contigs are shorter than 1000 bp and the task depends on reliable 4-mer frequency distances (F_4-mer is set to 1 there, so it carries no signal)
- Linking evidence is a single read pair per contig pair, since the scaffold graph requires at least two paired reads per link

## Inputs

- Assembled contigs (FASTA) from short-read assembly (e.g., MEGAHIT v1.2.3 initial contigs)
- Illumina paired-end short reads (FASTQ) for coverage depth and 4-mer estimation
- Paired long-range links (mate-pair/jumping library or long reads) mapped onto contigs (BWA/minimap2 alignments)

## Outputs

- Pruned scaffold graph with interspecies links excluded
- Table of retained/removed contig-pair links with posterior probabilities (e.g., misjoin_exclusion_results.tsv)
- Per-contig coverage depths and 4-mer frequency vectors used to build the empirical distributions

## How to apply

Build a scaffold graph whose edges connect contigs supported by at least two paired reads (mapped, e.g., with minimap2 or BWA as used by MetaPlatanus). For each linked contig pair, compute the coverage depth ratio min(c1,c2)/max(c1,c2) from short-read k-mer counts and the Euclidean distance between contig 4-mer frequency vectors. Estimate the same-species complementary cumulative distributions F_coverage and F_4-mer empirically from these quantities across all linked pairs. Exclude a link when the posterior score F_coverage(r) × F_4-mer(d) < 0.05; set F_4-mer to 1 for contigs shorter than 1000 bp since 4-mer frequencies are unreliable at that length. Prune flagged interspecies links and write retained/removed links with posterior probabilities (e.g., to misjoin_exclusion_results.tsv).

## Related tools

- **MetaPlatanus** (Implements the scaffold graph construction and the F_coverage × F_4-mer < 0.05 interspecies link-exclusion filter during scaffolding) — https://github.com/rkajitani/MetaPlatanus
- **minimap2** (Maps short-read pairs (preset -ax sr) onto contigs to quantify linkage supporting each contig pair) — https://github.com/lh3/minimap2
- **MEGAHIT** (Assembles the initial metagenomic contigs whose links are subsequently filtered) — https://github.com/voutcn/megahit

## Examples

```
minimap2 -ax sr contigs.fa reads_1.fq reads_2.fq > contig_links.sam
```

## Evaluation signals

- Planted cross-species links between simulated contigs with distinct coverage depths and oligonucleotide compositions are excluded from the scaffold graph
- Within-species links with matched coverage depth ratios and 4-mer profiles are retained in misjoin_exclusion_results.tsv
- Every excluded link satisfies the posterior criterion F_coverage(r) × F_4-mer(d) < 0.05
- Each retained scaffold-graph edge is supported by at least two linking read pairs
- F_4-mer equals 1 for all contig pairs involving a contig shorter than 1000 bp

## Limitations

- 4-mer frequencies are unreliable for contigs shorter than 1000 bp, so composition evidence is dropped (F_4-mer set to 1) for those pairs
- Requiring at least two paired reads per link trades sensitivity for safety, which matters for low-abundance species with few reads
- The empirical F_coverage and F_4-mer distributions are estimated from the same linked contig pairs being filtered, so heavily contaminated or coarse-grained assemblies may distort the null distributions

## Evidence
