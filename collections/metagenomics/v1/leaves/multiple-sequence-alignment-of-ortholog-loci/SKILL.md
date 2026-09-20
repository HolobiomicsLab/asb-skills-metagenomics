---
name: multiple-sequence-alignment-of-ortholog-loci
description: Use when you have recovered homologous single-copy marker sequences (e.g.
  USCOs/near-USCOs) for multiple specimens and need per-locus multiple sequence alignments
  as input for trimming, concatenation, and tree inference.
when_to_use_negative:
- The sequences are not homologous single-copy markers (e.g. variable-copy or chimeric
  sequences) — the article recommends limiting reference use to near-USCOs to avoid
  chimeric sequences
- Only a single sequence is available per locus, so no multiple sequence alignment
  can be computed
- The goal is marker retrieval from raw WGS reads itself — that is Patchwork's upstream
  role (DIAMOND search, hit stitching, sliding-window trimming), not the downstream
  MSA step
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Patchwork
  role: Upstream retrieval of the ortholog marker sequences that form the per-locus
    MSA input
  repo: https://github.com/fethalen/Patchwork
- name: MAFFT
  role: Builds the multiple sequence alignment for each of the 957 loci with --globalpair
    --ep 0.123
- name: trimAl
  role: Trims each MAFFT alignment with -gt 0.9 and -cons 60 before concatenation
- name: FASconCAT-G
  role: Concatenates the trimmed alignments into a single supermatrix and emits the
    gene partition file
- name: IQ-TREE 2
  role: Infers the maximum-likelihood phylogeny with -m MFP and 1,000 ultrafast bootstrap
    replicates from the partitioned supermatrix
- name: DIAMOND
  role: Sequence aligner used internally by Patchwork to obtain the homologous regions
    later aligned per locus
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:2f8edeaf87c5a34a93a0bece63dbc8ceacd7d0fe173acef435f98970b6576be9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - FASconCAT-G
  - FASconcat-G
  - IQ-TREE 2
  - MAFFT
  - Patchwork
  - trimAl
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Multiple sequence alignment of ortholog loci

## Summary

Aligns Patchwork-recovered ortholog marker sequences (e.g. 957 near-USCOs from H. robusta) per-locus with MAFFT using --globalpair --ep 0.123, so that the resulting alignments can be trimmed, concatenated, and used for maximum-likelihood phylogenomic tree inference.

## When to use

Use when you have recovered homologous single-copy marker sequences (e.g. USCOs/near-USCOs) for multiple specimens and need per-locus multiple sequence alignments as input for trimming, concatenation, and tree inference.

## When NOT to use

- The sequences are not homologous single-copy markers (e.g. variable-copy or chimeric sequences) — the article recommends limiting reference use to near-USCOs to avoid chimeric sequences
- Only a single sequence is available per locus, so no multiple sequence alignment can be computed
- The goal is marker retrieval from raw WGS reads itself — that is Patchwork's upstream role (DIAMOND search, hit stitching, sliding-window trimming), not the downstream MSA step

## Inputs

- Patchwork-recovered marker sequences per specimen, in FASTA format (one file or set of files per locus; 957 near-USCO loci from the H. robusta reference in the benchmark)
- Gene/partition definition for downstream concatenation (derived from the per-locus alignment boundaries)

## Outputs

- Per-locus MAFFT multiple sequence alignments (FASTA)
- Per-locus trimAl-trimmed alignments (FASTA)
- Trimmed concatenated supermatrix (3,454,320 positions retained, ~64% of untrimmed in the benchmark) plus a gene partition file from FASconCAT-G
- Maximum-likelihood phylogram from IQ-TREE 2 with 1,000 ultrafast bootstrap support values

## How to apply

Take the Patchwork-recovered marker sequences for each specimen as input and build one multiple sequence alignment per locus (957 loci in the louse benchmark) with MAFFT using the accurate iterative-refinement options --globalpair --ep 0.123. Trim each MAFFT alignment with trimAl using -gt 0.9 (remove positions with more than 90% gaps) and -cons 60 (retain at least 60% of each alignment). Concatenate the trimmed alignments into a single supermatrix with FASconCAT-G, keeping the corresponding gene partition file. Infer the phylogeny with IQ-TREE 2 under maximum likelihood using extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates (-B 1000). The rationale is that global-pair alignment maximizes accuracy for homologous ortholog loci, while the trimAl thresholds remove gappy, unreliable positions before supermatrix construction.

## Related tools

- **Patchwork** (Upstream retrieval of the ortholog marker sequences that form the per-locus MSA input) — https://github.com/fethalen/Patchwork
- **MAFFT** (Builds the multiple sequence alignment for each of the 957 loci with --globalpair --ep 0.123)
- **trimAl** (Trims each MAFFT alignment with -gt 0.9 and -cons 60 before concatenation)
- **FASconCAT-G** (Concatenates the trimmed alignments into a single supermatrix and emits the gene partition file)
- **IQ-TREE 2** (Infers the maximum-likelihood phylogeny with -m MFP and 1,000 ultrafast bootstrap replicates from the partitioned supermatrix)
- **DIAMOND** (Sequence aligner used internally by Patchwork to obtain the homologous regions later aligned per locus) — https://github.com/bbuchfink/diamond

## Examples

```
mafft --globalpair --ep 0.123 locus_001.fa > locus_001.aln.fasta && trimal -in locus_001.aln.fasta -out locus_001.trim.fasta -gt 0.9 -cons 60
```

## Evaluation signals

- Confirm the trimmed concatenated supermatrix and the gene partition file both exist after FASconCAT-G
- Confirm the IQ-TREE 2 run completed 1,000 ultrafast bootstrap replicates (-B 1000) and produced a well-supported tree
- In the benchmark, the trimmed supermatrix retained 3,454,320 positions (~64% of the untrimmed length) — large deviations suggest misapplied trimAl thresholds
- Resulting tree should be largely congruent with the original (published) louse phylogeny for the phthirapteran data set
- Each of the 957 loci should yield a non-empty trimmed alignment before concatenation

## Limitations

- Reference sequences should be limited to near-USCOs; using other references risks chimeric sequences entering the alignments
- Residual noncoding or artifactual stretches may survive; the article suggests PREQUAL could be used to detect and remove such regions
- The skill was validated on a phthirapteran data set against 957 near-USCOs from H. robusta; transfer to other taxonomic scales may need re-tuned trimming parameters

## Evidence

- [full_text] MAFFT globalpair alignment of 957 loci: [evidence span withheld — anchor status: not_found]
- [full_text] trimAl trimming thresholds: [evidence span withheld — anchor status: not_found]
- [full_text] IQ-TREE 2 inference settings: [evidence span withheld — anchor status: not_found]
- [results] Well-supported reconstruction from retrieved markers: "Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6)"
- [readme] Patchwork produces the stitched marker sequences used for alignment: "Stitch overlapping or gappy sequences together based on a reference"
