---
name: ultrafast-bootstrap-support-assessment
description: Use when a trimmed concatenated alignment and gene partition file (e.g.,
  FASconCAT-G output from 957 Patchwork-recovered USCO loci) are ready and node-support
  statistics are needed for a maximum-likelihood tree.
when_to_use_negative:
- Input is a set of raw Patchwork hit fragments or untrimmed MAFFT alignments rather
  than the concatenated supermatrix and partition file — alignment, trimming, and
  concatenation (MAFFT, trimAl, FASconCAT-G) must precede tree inference
- The goal is marker retrieval quality assessment (identity/coverage benchmarking
  against ALiBaSeq or aTRAM 2) rather than phylogenetic inference
edam_operation: http://edamontology.org/operation_0266
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0684
tools:
- name: IQ-TREE 2
  role: Maximum-likelihood tree inference with extended model selection (-m MFP) and
    1,000 ultrafast bootstrap replicates (-B 1000) on the partitioned supermatrix
- name: FASconCAT-G
  role: Concatenates the 957 trimmed MAFFT/trimAl alignments into the single supermatrix
    and emits the gene partition file consumed by IQ-TREE 2
- name: MAFFT
  role: Builds the per-locus multiple sequence alignments (--globalpair --ep 0.123)
    that are trimmed and concatenated before tree inference
- name: trimAl
  role: Trims each MAFFT alignment (-gt 0.9, -cons 60) to remove gap-heavy positions
    prior to concatenation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:d9b2070ccd065e853befa62698a0e6a423c82d26d0f543d80a2b1c04a14b1932
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FASconCAT-G
  - FASconcat-G
  - IQ-TREE 2
  - MAFFT
  - Patchwork
  - trimAl
  edam_operation: http://edamontology.org/operation_0266
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0684
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ultrafast-bootstrap-support-assessment

## Summary

Infers a maximum-likelihood phylogeny from the trimmed, concatenated Patchwork-recovered USCO supermatrix using IQ-TREE 2 with extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates (-B 1000), producing a well-supported tree whose support values can be assessed against a reference phylogeny.

## When to use

Use when a trimmed concatenated alignment and gene partition file (e.g., FASconCAT-G output from 957 Patchwork-recovered USCO loci) are ready and node-support statistics are needed for a maximum-likelihood tree.

## When NOT to use

- Input is a set of raw Patchwork hit fragments or untrimmed MAFFT alignments rather than the concatenated supermatrix and partition file — alignment, trimming, and concatenation (MAFFT, trimAl, FASconCAT-G) must precede tree inference
- The goal is marker retrieval quality assessment (identity/coverage benchmarking against ALiBaSeq or aTRAM 2) rather than phylogenetic inference

## Inputs

- Concatenated supermatrix (e.g., FASTA, 3,454,320 retained positions after trimAl) produced by FASconCAT-G
- Gene partition file mapping each of the 957 loci to alignment coordinates, produced by FASconCAT-G
- Per-locus trimmed MAFFT/trimAl alignments (trimAl -gt 0.9, -cons 60) as upstream input to concatenation

## Outputs

- Maximum-likelihood phylogenetic tree (Newick) with ultrafast bootstrap support values on nodes
- IQ-TREE 2 partition/model report with best-fit substitution models per gene partition
- Ultrafast bootstrap consensus/support files (e.g., treefile, .contree, .splits.nex) from the 1,000 replicates

## How to apply

Take the FASconCAT-G concatenated supermatrix and its corresponding gene partition file as input to IQ-TREE 2 under maximum likelihood. Enable extended model selection with -m MFP so each partition can receive its own best-fit substitution model, and run 1,000 ultrafast bootstrap replicates with -B 1000 to obtain node support. Confirm the run completed all 1,000 ultrafast bootstrap replicates and that the supermatrix and partition file were correctly supplied. Judge success by whether the resulting tree is well supported and largely congruent with the original louse phylogeny (Allen et al. 2017), as reported for the 3,454,320-position trimmed supermatrix.

## Related tools

- **IQ-TREE 2** (Maximum-likelihood tree inference with extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates (-B 1000) on the partitioned supermatrix)
- **FASconCAT-G** (Concatenates the 957 trimmed MAFFT/trimAl alignments into the single supermatrix and emits the gene partition file consumed by IQ-TREE 2)
- **MAFFT** (Builds the per-locus multiple sequence alignments (--globalpair --ep 0.123) that are trimmed and concatenated before tree inference)
- **trimAl** (Trims each MAFFT alignment (-gt 0.9, -cons 60) to remove gap-heavy positions prior to concatenation)

## Examples

```
iqtree2 -s supermatrix.fasta -p partition.nex -m MFP -B 1000
```

## Evaluation signals

- The concatenated supermatrix and gene partition file both exist and are consistent in sequence count before IQ-TREE 2 is launched
- The IQ-TREE 2 log confirms all 1,000 ultrafast bootstrap replicates completed without error
- The output tree is well supported, with ultrafast bootstrap values largely congruent with the original phthirapteran (louse) phylogeny of Allen et al. 2017
- The retained supermatrix length matches the trimAl-filtered expectation (~3,454,320 positions, ~64% of the untrimmed alignment)
- Best-fit substitution models are reported per partition (as expected from -m MFP), not a single global model

## Limitations

- Tree support is conditional on upstream quality: most markers must clear the 90% aligned-position trimming threshold, and recovered sequences from distant references reached only ~89% average percent identity and 74.5% mean query coverage
- Ultrafast bootstrap support reflects resampling of the concatenated signal and cannot compensate for chimeric or non-single-copy sequences; the article recommends limiting references to near-USCOs to avoid chimeras
- Reference sequences far from the query taxon reduce recovery (943 of 957 H. robusta near-USCOs retrieved from D. gyrociliatus), which thins the supermatrix and can weaken support for some nodes

## Evidence

- [other] IQ-TREE 2 under maximum likelihood using extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates (-B 1000): [evidence span withheld — anchor status: not_found]
- [results] a trimmed concatenated alignment (3,454,320 positions retained, ~64% of untrimmed) and a well-supported tree largely congruent with the original louse phylogeny: [evidence span withheld — anchor status: not_found]
- [results] Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6): "Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6)"
- [other] confirm that the concatenated supermatrix and gene partition file exist, and that the IQ-TREE 2 run completed 1,000 ultrafast bootstrap replicates: [evidence span withheld — anchor status: not_found]
