---
name: supermatrix-concatenation-phylogenomics
description: Use when you already hold recovered marker sequences (e.g., FASTA per
  specimen per locus) for multiple specimens and need to align, trim, concatenate
  them into a supermatrix, and infer a maximum-likelihood phylogeny.
when_to_use_negative:
- Markers are not (putatively) single-copy orthologs shared across specimens (e.g.,
  multi-copy or non-homologous sequences), as concatenation assumes locus equivalence
  across taxa
- You have only one or very few loci, where a supermatrix/partitioned approach adds
  no benefit over single-locus phylogenetics
- Sequences still contain unaligned residues, stop codons, or artifactual noncoding
  stretches that have not been cleaned (e.g., via Patchwork's alignment masking or
  PREQUAL), which would corrupt the MAFFT/trimAl step
edam_operation: http://edamontology.org/operation_0554
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: MAFFT
  role: Builds the multiple sequence alignment for each of the 957 loci with --globalpair
    --ep 0.123
- name: trimAl
  role: Trims each MAFFT alignment using -gt 0.9 (remove positions with more than
    90% gaps) and -cons 60 (retain at least 60% of each alignment)
- name: FASconCAT-G
  role: Concatenates the trimmed per-locus alignments into a single supermatrix and
    produces the gene partition file
- name: IQ-TREE 2
  role: Infers the maximum-likelihood phylogeny from the supermatrix using -m MFP
    extended model selection and 1,000 ultrafast bootstrap replicates (-B 1000)
- name: Patchwork
  role: Upstream source of the recovered marker sequences fed into the downstream
    pipeline
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:63aca57f54d6778d5bb9f5f6f27e128f45d4c412dcb7857de5e1772612a97743
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - FASconCAT-G
  - FASconcat-G
  - IQ-TREE 2
  - MAFFT
  - Patchwork
  - trimAl
  edam_operation: http://edamontology.org/operation_0554
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# supermatrix-concatenation-phylogenomics

## Summary

Carries marker sequences (e.g., Patchwork-recovered USCOs) through a downstream phylogenomic pipeline: per-locus MAFFT alignment, trimAl trimming, FASconCAT-G concatenation with a gene partition file, and IQ-TREE 2 maximum-likelihood tree inference with ultrafast bootstraps. It tests whether retrieved markers can support a well-resolved, well-supported phylogeny.

## When to use

Use when you already hold recovered marker sequences (e.g., FASTA per specimen per locus) for multiple specimens and need to align, trim, concatenate them into a supermatrix, and infer a maximum-likelihood phylogeny.

## When NOT to use

- Markers are not (putatively) single-copy orthologs shared across specimens (e.g., multi-copy or non-homologous sequences), as concatenation assumes locus equivalence across taxa
- You have only one or very few loci, where a supermatrix/partitioned approach adds no benefit over single-locus phylogenetics
- Sequences still contain unaligned residues, stop codons, or artifactual noncoding stretches that have not been cleaned (e.g., via Patchwork's alignment masking or PREQUAL), which would corrupt the MAFFT/trimAl step

## Inputs

- Patchwork-recovered marker sequences per specimen (FASTA) for each of the 957 near-USCO loci
- Mapping of specimens to loci/recovered marker IDs

## Outputs

- Per-locus MAFFT alignments (untrimmed FASTA)
- Per-locus trimAl-trimmed alignments (FASTA)
- Concatenated supermatrix (FASTA/phylip) with corresponding gene partition file from FASconCAT-G (e.g., 3,454,320 positions retained, ~64% of untrimmed)
- IQ-TREE 2 maximum-likelihood tree with 1,000 ultrafast bootstrap support values (Newick plus .iqtree log files)

## How to apply

Take Patchwork-recovered marker sequences per specimen (here, USCOs queried against 957 near-USCOs from H. robusta) and build a multiple sequence alignment for each of the 957 loci with MAFFT using --globalpair --ep 0.123. Trim each MAFFT alignment with trimAl using -gt 0.9 (remove positions with more than 90% gaps) and -cons 60 (retain at least 60% of each alignment). Concatenate the trimmed alignments into a single supermatrix with FASconCAT-G, retaining the corresponding gene partition file. Infer the phylogeny with IQ-TREE 2 under maximum likelihood using extended model selection (-m MFP) and 1,000 ultrafast bootstrap replicates (-B 1000), supplying the supermatrix and its partition file. Rationale: consistent per-locus alignment and gap-based trimming removes poorly aligned or gappy positions before concatenation, and the partition file lets IQ-TREE 2 select a separate model per gene.

## Related tools

- **MAFFT** (Builds the multiple sequence alignment for each of the 957 loci with --globalpair --ep 0.123)
- **trimAl** (Trims each MAFFT alignment using -gt 0.9 (remove positions with more than 90% gaps) and -cons 60 (retain at least 60% of each alignment))
- **FASconCAT-G** (Concatenates the trimmed per-locus alignments into a single supermatrix and produces the gene partition file)
- **IQ-TREE 2** (Infers the maximum-likelihood phylogeny from the supermatrix using -m MFP extended model selection and 1,000 ultrafast bootstrap replicates (-B 1000))
- **Patchwork** (Upstream source of the recovered marker sequences fed into the downstream pipeline) — https://github.com/fethalen/Patchwork

## Examples

```
iqtree2 -s supermatrix.fasta -p partition_file.nex -m MFP -B 1000
```

## Evaluation signals

- The concatenated supermatrix and gene partition file both exist, with the partition boundaries consistent with the trimmed per-locus alignments
- The IQ-TREE 2 run completed 1,000 ultrafast bootstrap replicates and produced a well-supported tree (e.g., the reconstructed tree was largely congruent with the original louse phylogeny)
- Trimming retained a substantial fraction of the untrimmed matrix (e.g., 3,454,320 positions, ~64% of the untrimmed alignment), and most markers were well above the 90% aligned position trimming threshold
- Per-locus alignments cover the expected number of loci (957) and specimens without missing specimen files
- The resulting tree's topology and bootstrap support are compared against a reference phylogeny for congruence

## Limitations

- Results depend on upstream marker retrieval quality: distant references (e.g., H. robusta near-USCOs applied to D. gyrociliatus) recovered 943 of 957 sequences, and only 769 aligned back uniquely, so missing or ambiguously recovered loci propagate into the supermatrix
- trimAl parameters (-gt 0.9, -cons 60) can discard loci or positions with low coverage; markers near the thresholds may drop out entirely
- Limiting reference sequences to near-USCOs is recommended to avoid chimeric sequences, which would distort concatenated alignments

## Evidence

- [results] Running the downstream workflow on the Patchwork-recovered USCOs yielded a trimmed concatenated alignment (3,454,320 positions retained, ~64% of untrimmed): [evidence span withheld — anchor status: not_found]
- [methods] MAFFT alignment step: [evidence span withheld — anchor status: not_found]
- [methods] trimAl trimming parameters: [evidence span withheld — anchor status: not_found]
- [methods] IQ-TREE 2 tree inference settings: [evidence span withheld — anchor status: not_found]
- [results] most markers were well above the 90% aligned position trimming threshold: "most markers were well above the 90% aligned position trimming threshold"
- [results] Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6): "Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6)"
