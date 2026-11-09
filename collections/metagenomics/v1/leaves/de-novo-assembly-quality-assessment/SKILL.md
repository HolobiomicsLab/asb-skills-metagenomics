---
name: de-novo-assembly-quality-assessment
description: Use when subsampled low-coverage WGS read sets (e.g. 1x–40x) must be
  assembled de novo to create query contigs, and assembly quality must be verified
  before marker retrieval.
when_to_use_negative:
- Coverage is 1x or otherwise extremely low — the article found read-based (aTRAM/Patchwork
  on reads) marker retrieval outperformed assembly-based retrieval at 1x coverage
- A high-quality reference genome exists and reference-guided assembly is preferred
  over de novo contiguity
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: SPAdes
  role: De novo assembly of the subsampled short-read data sets (K-mer size 33) to
    produce query contigs
- name: QUAST
  role: Quality assessment of each coverage-arm assembly before marker retrieval
- name: Subsample.jl
  role: Random selection of corresponding read pairs to generate 1x–40x coverage data
    sets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:58a4deb206e5a0d5ef03667b3cd8f8ea78db108c6b6ad71628a8de386bc641a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALiBaSeq
  - BLAST
  - Julia
  - Patchwork
  - QUAST
  - SPAdes
  - Slurm
  - Subsample.jl
  - aTRAM 2
  - exonerate
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# De novo assembly and quality assessment of subsampled short-read data

## Summary

Build short-read-only de novo assemblies (SPAdes, K-mer size 33) from subsampled paired-end Illumina data and assess assembly quality with QUAST before using the contigs as Patchwork/ALiBaSeq query input.

## When to use

Use when subsampled low-coverage WGS read sets (e.g. 1x–40x) must be assembled de novo to create query contigs, and assembly quality must be verified before marker retrieval.

## When NOT to use

- Coverage is 1x or otherwise extremely low — the article found read-based (aTRAM/Patchwork on reads) marker retrieval outperformed assembly-based retrieval at 1x coverage
- A high-quality reference genome exists and reference-guided assembly is preferred over de novo contiguity

## Inputs

- Trimmed paired-end Illumina short reads subsampled to target coverage (1x, 3x, 5x, 10x, 20x, 40x; FASTQ)
- Read-pair correspondence metadata for paired subsampling

## Outputs

- Per-coverage-arm de novo assembly contigs (SPAdes, K-mer 33)
- QUAST assembly quality reports per coverage arm
- Contig FASTA files suitable as query input for Patchwork/ALiBaSeq

## How to apply

Subsample the paired-end reads (e.g. D. gyrociliatus Illumina reads, PRJEB37657) to defined coverage arms — 1x, 3x, 5x, 10x, 20x, 40x — by randomly selecting corresponding read pairs with a tool such as Subsample.jl. Assemble each arm de novo with SPAdes using K-mer size 33, keeping the assemblies short-read-only. Assess each assembly with QUAST to obtain contiguity and length statistics, and use these metrics as a decision point: at very low coverage (1x) the article found read-based marker retrieval outperformed assembly-based retrieval, so assembly quality determines whether contigs are a suitable Patchwork/ALiBaSeq input. Feed the assessed contigs downstream to alignment-based marker retrieval against protein references (e.g. 957 H. robusta near-USCOs).

## Related tools

- **SPAdes** (De novo assembly of the subsampled short-read data sets (K-mer size 33) to produce query contigs)
- **QUAST** (Quality assessment of each coverage-arm assembly before marker retrieval)
- **Subsample.jl** (Random selection of corresponding read pairs to generate 1x–40x coverage data sets)

## Examples

```
spades.py -1 reads_10x_R1.fq -2 reads_10x_R2.fq -k 33 -o spades_10x && quast.py spades_10x/contigs.fasta -o quast_10x
```

## Evaluation signals

- QUAST report is generated for every coverage arm (1x, 3x, 5x, 10x, 20x, 40x) with contig statistics present
- Each subsampled FASTQ pair contains matching forward/reverse reads (corresponding read pairs preserved by Subsample.jl)
- Assembly contiguity improves with coverage; the 1x arm is expected to be highly fragmented, consistent with the article's finding that read-based retrieval beats assembly-based retrieval at 1x
- Downstream Patchwork with assembly input reaches up to 62% total percent identity only for arms with >=10x coverage — assemblies from lower arms should yield weaker marker recovery

## Limitations

- Assembly-based marker retrieval degrades sharply below 10x coverage; at 1x, read-based data are better suited for marker retrieval
- Highly fragmented low-coverage assemblies limit the hit-stitching capability of downstream alignment tools

## Evidence

- [results] Subsampled reads assembled with SPAdes before Patchwork marker retrieval: "We assembled these sequence reads using SPAdes and subsequently used Patchwork to search for near-USCOs"
- [results] Subsampling to simulate coverage arms: "we also subsampled the aforementioned D. gyrociliatus short reads in order to simulate various sequencing coverages"
- [results] Assembly data performs best only at adequate coverage: "Patchwork with D. gyrociliatus assembly data performs best over almost all data sets (Fig. 4), reaching as much as 62% total percent identity for data with at least 10× coverage"
- [results] At 1x coverage reads outperform assemblies: "Only with a coverage of 1×, D. gyrociliatus read-based data seem to be better suited for marker retrieval"
