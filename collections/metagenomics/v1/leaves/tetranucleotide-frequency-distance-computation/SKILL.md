---
name: tetranucleotide-frequency-distance-computation
description: Use when scaffolding metagenomic contigs with paired long-range links
  and you must decide which links are within-species versus probable interspecies
  misjoins, using coverage depth ratios and 4-mer frequency distributions.
when_to_use_negative:
- Contigs shorter than 1000 bp are involved, since F_4-mer is forced to 1 due to unreliable
  4-mer frequencies on short sequences
- Input is a single-isolate assembly rather than a mixed metagenome, where interspecies
  misjoin filtering is not the failure mode of concern
- No paired-end short-read library is available, which MetaPlatanus requires at minimum
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Implements the scaffold graph with the coverage-depth-ratio and 4-mer empirical-distribution
    exclusion filter (F_coverage × F_4-mer < 0.05)
  repo: https://github.com/rkajitani/MetaPlatanus
- name: minimap2
  role: Maps paired short reads onto contigs to identify contig pairs supported by
    at least two linking read pairs
  repo: https://github.com/lh3/minimap2
- name: MEGAHIT
  role: Assembles the initial contigs whose links are then evaluated by the exclusion
    filter
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:5c033050c207ce18dfa5e88ec2e33d6c77e908b034de2a69012c32963717447c
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# tetranucleotide-frequency-distance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise distances between contig 4-mer (tetranucleotide) frequency vectors and combines them with contig-pair coverage-depth ratios to exclude probable interspecies misjoinings in MetaPlatanus-style metagenome scaffolding.

## When to use

Use when scaffolding metagenomic contigs with paired long-range links and you must decide which links are within-species versus probable interspecies misjoins, using coverage depth ratios and 4-mer frequency distributions.

## When NOT to use

- Contigs shorter than 1000 bp are involved, since F_4-mer is forced to 1 due to unreliable 4-mer frequencies on short sequences
- Input is a single-isolate assembly rather than a mixed metagenome, where interspecies misjoin filtering is not the failure mode of concern
- No paired-end short-read library is available, which MetaPlatanus requires at minimum

## Inputs

- Contig FASTA from short-read contig assembly (e.g. MEGAHIT output)
- Paired-end short-read FASTQ (one library required)
- Short-read alignments of read pairs onto contigs (SAM/PAF from minimap2 or BWA)
- Per-contig k-mer coverage depths and 4-mer frequency vectors

## Outputs

- misjoin_exclusion_results.tsv with retained/removed links and posterior probabilities
- Pruned scaffold graph containing only links passing F_coverage × F_4-mer >= 0.05

## How to apply

Build a scaffold graph whose edges join contigs supported by at least two paired reads; map short reads to contigs with an aligner such as minimap2 or BWA as used by MetaPlatanus. For each linked contig pair compute the coverage depth ratio min(c1,c2)/max(c1,c2) and the Euclidean distance between the contigs' 4-mer frequency vectors. Empirically estimate the same-species complementary cumulative distributions F_coverage and F_4-mer from these quantities, setting F_4-mer to 1 for contigs shorter than 1000 bp because they are too short for reliable 4-mer frequencies. Exclude a link when the combined posterior score F_coverage(r) × F_4-mer(d) < 0.05, i.e. when the link is unlikely under the same-species null distributions. Retain the surviving links for scaffolding and record removed links with their posterior probabilities.

## Related tools

- **MetaPlatanus** (Implements the scaffold graph with the coverage-depth-ratio and 4-mer empirical-distribution exclusion filter (F_coverage × F_4-mer < 0.05)) — https://github.com/rkajitani/MetaPlatanus
- **minimap2** (Maps paired short reads onto contigs to identify contig pairs supported by at least two linking read pairs) — https://github.com/lh3/minimap2
- **MEGAHIT** (Assembles the initial contigs whose links are then evaluated by the exclusion filter) — https://github.com/voutcn/megahit

## Examples

```
minimap2 -ax sr contigs.fa reads_1.fq reads_2.fq > aln.sam
```

## Evaluation signals

- Links planted between simulated cross-species contigs are excluded while within-species links are retained in misjoin_exclusion_results.tsv
- Every flagged link satisfies F_coverage(r) × F_4-mer(d) < 0.05 under the empirically estimated same-species complementary cumulative distributions
- All contig pairs shorter than 1000 bp carry F_4-mer = 1 in the computation
- Each edge in the scaffold graph is supported by at least two paired reads mapped to the contig pair
- Coverage depth ratio min(c1,c2)/max(c1,c2) is finite and in (0,1] for every evaluated link

## Limitations

- 4-mer frequencies are unreliable for contigs shorter than 1000 bp, so the filter falls back to coverage depth alone (F_4-mer set to 1) for such contigs
- Low-abundance species yield few links, motivating the design goal of joining sequences with a small number of links while avoiding misjoins
- The distributions are estimated empirically from the same dataset, so heavily skewed community compositions may distort the same-species null

## Evidence
