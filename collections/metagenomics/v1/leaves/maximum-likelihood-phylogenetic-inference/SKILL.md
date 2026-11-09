---
name: maximum-likelihood-phylogenetic-inference
description: Use when you already have per-locus marker sequences (e.g. Patchwork-recovered
  near-USCOs) and need a well-supported phylogenomic tree via alignment, trimming,
  concatenation, and ML tree inference.
when_to_use_negative:
- When marker sequences have not yet been retrieved/homology-screened (e.g. raw WGS
  reads or contigs lacking Patchwork DIAMOND search, hit stitching, and sliding-window
  trimming) — this skill starts from recovered per-locus sequences.
- When the question calls for the marker-retrieval benchmark itself rather than phylogeny
  construction from recovered markers.
edam_operation: http://edamontology.org/operation_0554
edam_topics: []
tools:
- name: Patchwork
  role: source of recovered marker (near-USCO) sequences that serve as the input for
    the phylogenomic pipeline
  repo: https://github.com/fethalen/Patchwork
- name: MAFFT
  role: builds per-locus multiple sequence alignments with --globalpair --ep 0.123
- name: trimAl
  role: trims MAFFT alignments using -gt 0.9 and -cons 60
- name: FASconcat-G
  role: concatenates trimmed alignments into a supermatrix with a gene partition file
- name: IQ-TREE 2
  role: maximum-likelihood tree inference with -m MFP and -B 1000 ultrafast bootstraps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:07c5826a47d4a38ecc458b2ab0210a89b6f5fc55dc8933709f0469b7c294a851
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - BLAST
  - FASconcat-G
  - FastTree
  - IQ-TREE
  - IQ-TREE 2
  - Infernal
  - MAFFT
  - MAGinator
  - NCBI RefSeq
  - Patchwork
  - Prodigal
  - RAxML
  - SSUAlign
  - Samtools
  - Snakemake
  - seqkit
  - trimAl
  edam_operation: http://edamontology.org/operation_0554
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# maximum-likelihood-phylogenetic-inference

## Summary

Infer a maximum-likelihood phylogeny from a concatenated supermatrix of trimmed multiple sequence alignments using IQ-TREE 2 with extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates, as applied downstream of Patchwork-recovered USCO markers.

## When to use

Use when you already have per-locus marker sequences (e.g. Patchwork-recovered near-USCOs) and need a well-supported phylogenomic tree via alignment, trimming, concatenation, and ML tree inference.

## When NOT to use

- When marker sequences have not yet been retrieved/homology-screened (e.g. raw WGS reads or contigs lacking Patchwork DIAMOND search, hit stitching, and sliding-window trimming) — this skill starts from recovered per-locus sequences.
- When the question calls for the marker-retrieval benchmark itself rather than phylogeny construction from recovered markers.

## Inputs

- Patchwork-recovered marker sequences for each specimen (per-locus nucleotide FASTA, from 957 near-USCOs of H. robusta)

## Outputs

- Per-locus MAFFT multiple sequence alignments
- trimAl-trimmed per-locus alignments (3,454,320 positions retained in the concatenated matrix, ~64% of untrimmed)
- Concatenated supermatrix plus gene partition file (FASconCAT-G)
- Maximum-likelihood phylogenetic tree with ultrafast bootstrap support values (IQ-TREE 2)

## How to apply

Take the Patchwork-recovered marker sequences for each specimen (queried against 957 near-USCOs from H. robusta) as input. Align each of the 957 loci with MAFFT using --globalpair --ep 0.123, then trim with trimAl using -gt 0.9 (positions with >90% gaps removed) and -cons 60 (retain at least 60% of each alignment). Concatenate the trimmed alignments into a single supermatrix with FASconCAT-G, keeping the corresponding gene partition file. Finally, infer the phylogeny with IQ-TREE 2 under maximum likelihood with extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates (-B 1000), supplying the supermatrix and its partition file.

## Related tools

- **Patchwork** (source of recovered marker (near-USCO) sequences that serve as the input for the phylogenomic pipeline) — https://github.com/fethalen/Patchwork
- **MAFFT** (builds per-locus multiple sequence alignments with --globalpair --ep 0.123)
- **trimAl** (trims MAFFT alignments using -gt 0.9 and -cons 60)
- **FASconcat-G** (concatenates trimmed alignments into a supermatrix with a gene partition file)
- **IQ-TREE 2** (maximum-likelihood tree inference with -m MFP and -B 1000 ultrafast bootstraps)

## Examples

```
iqtree2 -s supermatrix.fasta -p partition.txt -m MFP -B 1000
```

## Evaluation signals

- The concatenated supermatrix and the gene partition file both exist after the FASconCAT-G step.
- The trimmed concatenated alignment retains a large fraction of untrimmed positions (in the article ~64%, i.e. 3,454,320 of the positions were retained with trimAl -gt 0.9 -cons 60).
- The IQ-TREE 2 run completed the full 1,000 ultrafast bootstrap replicates without error.
- The resulting tree is well-supported (high ultrafast bootstrap values) and largely congruent with the original louse phylogeny used as the reference.

## Limitations

- Alignment trimming removes a substantial fraction of positions (in the article ~36% of positions were removed), so the final supermatrix covers only part of the untrimmed length.
- Tree quality depends on the completeness and identity of the recovered markers; against distant references Patchwork retrieved 943 of 957 sequences with 89% average percent identity and 74.5% mean query coverage, so missing or divergent loci can affect inference.
- The article recommends limiting reference sequences to near-USCOs to avoid chimeric sequences, which could otherwise propagate into the alignment and tree.

## Evidence

- [other] MAFFT --globalpair --ep 0.123: [evidence span withheld — anchor status: not_found]
- [other] trimAl parameters: [evidence span withheld — anchor status: not_found]
- [other] IQ-TREE 2 invocation: [evidence span withheld — anchor status: not_found]
- [results] tree result: "Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6)"
- [readme] README features: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
