---
name: phylogenetic-tree-inference
description: Use when you have a FASTA of de-novo clustered OTU representative sequences
  (e.g. from micca otu) and need a Newick phylogenetic tree with one leaf per OTU
  for downstream diversity or ecology analyses.
when_to_use_negative:
- Input is raw amplicon reads rather than clustered OTU representative sequences —
  quality filtering, mergepairs, and OTU picking must run first
- The analysis only needs taxonomy assignment (RDP classifier) and no phylogeny-aware
  diversity metrics
- Reference-based tree placement against a curated database is required rather than
  de novo FastTree inference on the study's own OTUs
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3697
tools:
- name: MICCA
  role: Pipeline wrapper providing the 'msa' and 'tree' commands with a fast, memory-efficient
    NAST multiple sequence alignment module
  repo: https://github.com/compmetagen/micca
- name: MUSCLE
  role: Computes the multiple sequence alignment of OTU representative sequences inside
    both micca msa and micca tree
- name: FastTree
  role: Infers the phylogenetic tree from the aligned OTU representative sequences
    inside micca tree
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:e2c82646a77dd28a32a4188c65e159aa814079b7a35ccefb2a46d8f7fe6cd33b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - FastTree
  - MICCA
  - MUSCLE
  - RDP classifier
  - VSEARCH
  - swarm
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# phylogenetic-tree-inference

## Summary

Build a multiple sequence alignment and phylogenetic tree from OTU representative sequences using MICCA's MSA and tree commands, which invoke MUSCLE and FastTree, as part of the MICCA amplicon taxonomic profiling pipeline.

## When to use

Use when you have a FASTA of de-novo clustered OTU representative sequences (e.g. from micca otu) and need a Newick phylogenetic tree with one leaf per OTU for downstream diversity or ecology analyses.

## When NOT to use

- Input is raw amplicon reads rather than clustered OTU representative sequences — quality filtering, mergepairs, and OTU picking must run first
- The analysis only needs taxonomy assignment (RDP classifier) and no phylogeny-aware diversity metrics
- Reference-based tree placement against a curated database is required rather than de novo FastTree inference on the study's own OTUs

## Inputs

- OTU representative-sequence FASTA file produced by the de novo OTU clustering step (micca otu)

## Outputs

- Multiple sequence alignment of OTU representative sequences (MUSCLE-based, via micca msa)
- Phylogenetic tree in Newick format (tree.nwk) with one leaf per OTU representative (FastTree, via micca tree)

## How to apply

Load the OTU representative-sequence FASTA produced by the de novo clustering step (micca otu, 97% identity). Run the MICCA 'msa' command, which invokes MUSCLE to compute a multiple sequence alignment of the representative sequences. Then run the MICCA 'tree' command, which again uses MUSCLE for the alignment and FastTree to infer the phylogenetic tree, leveraging MICCA's fast and memory-efficient NAST MSA module. Validate that tree.nwk exists, parses as a Newick-format tree, and contains exactly one leaf per input OTU representative sequence. If used in a publication, cite the wrapped third-party tools (MUSCLE, FastTree, VSEARCH) alongside MICCA itself.

## Related tools

- **MICCA** (Pipeline wrapper providing the 'msa' and 'tree' commands with a fast, memory-efficient NAST multiple sequence alignment module) — https://github.com/compmetagen/micca
- **MUSCLE** (Computes the multiple sequence alignment of OTU representative sequences inside both micca msa and micca tree)
- **FastTree** (Infers the phylogenetic tree from the aligned OTU representative sequences inside micca tree)

## Examples

```
micca msa -i otu.fasta -o msa.fasta; micca tree -i msa.fasta -o tree.nwk
```

## Evaluation signals

- tree.nwk exists and parses as a valid Newick-format tree
- Number of leaves in the tree equals the number of OTU representative sequences supplied as input (one leaf per OTU)
- An intermediate MSA of representative sequences was produced by MUSCLE before tree inference
- In the antibiotic time-series reproduction, the tree can be built from the ~1466 OTU representatives obtained from ~3.2e6 merged reads per sample

## Limitations

- FastTree inference is de novo on the study's own OTU representatives; no reference-based phylogenetic placement is performed
- MICCA wraps third-party packages (MUSCLE, FastTree, VSEARCH) which should be cited if used
- Pipeline results depend on upstream quality filtering (expected error criterion) and clustering parameters; poor OTU input yields a poor tree

## Evidence

- [other] MICCA implements the phylogeny step through a fast and memory efficient NAST multiple sequence alignment (MSA) module followed by phylogenetic tree inference: [evidence span withheld — anchor status: not_found]
- [readme] fast and and memory efficient **NAST** multiple sequence alignment (MSA);: "fast and and memory efficient **NAST** multiple sequence alignment (MSA);"
- [readme] MUSCLE used in msa and tree commands: "* MUSCLE (doi: 10.1093/nar/gkh340) used in ``msa`` and ``tree`` commands"
- [readme] FastTree used in the tree command: "* FastTree (doi: 10.1371/journal.pone.0009490) used in the ``tree`` command"
