---
name: amplicon-metagenomic-workflow-command-execution
description: Use when you have an OTU representative-sequence FASTA file from MICCA
  de novo clustering and need an aligned set and phylogenetic tree (Newick) for downstream
  phylogenetic diversity metrics.
when_to_use_negative:
- Input is raw reads or a sample-by-OTU abundance table rather than an OTU representative-sequence
  FASTA — the msa/tree commands operate on sequences from the pipeline's clustering
  step
- You need taxonomy assignment rather than a phylogeny — use MICCA's 'classify' command
  (RDP classifier) instead
- Sequences are unrelated or non-homologous such that a NAST alignment against a common
  backbone is meaningless
edam_operation: http://edamontology.org/operation_0557
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0637
tools:
- name: MICCA
  role: Pipeline providing the 'msa' and 'tree' commands for alignment and phylogenetic
    inference
  repo: https://github.com/compmetagen/micca
- name: MUSCLE
  role: Underlying aligner invoked by both the micca 'msa' and 'tree' commands
- name: FastTree
  role: Phylogenetic tree inference engine invoked by the micca 'tree' command
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:d107edfe7d13c31b8fcfffb05fa13ca7dd74f16f032e0861797dfdd2bd0461fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - FastTree
  - MICCA
  - MUSCLE
  edam_operation: http://edamontology.org/operation_0557
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# amplicon-metagenomic-workflow-command-execution

## Summary

Reconstructs the MICCA multiple-sequence alignment (MSA) and FastTree phylogenetic inference step for OTU representative sequences, closing the amplicon pipeline from clustered OTUs to a Newick tree usable for diversity analysis.

## When to use

Use when you have an OTU representative-sequence FASTA file from MICCA de novo clustering and need an aligned set and phylogenetic tree (Newick) for downstream phylogenetic diversity metrics.

## When NOT to use

- Input is raw reads or a sample-by-OTU abundance table rather than an OTU representative-sequence FASTA — the msa/tree commands operate on sequences from the pipeline's clustering step
- You need taxonomy assignment rather than a phylogeny — use MICCA's 'classify' command (RDP classifier) instead
- Sequences are unrelated or non-homologous such that a NAST alignment against a common backbone is meaningless

## Inputs

- FASTA file of OTU representative sequences (output of MICCA de novo OTU clustering)

## Outputs

- Multiple sequence alignment of OTU representative sequences (from the micca msa command)
- Newick-format phylogenetic tree file (tree.nwk) with one leaf per OTU representative sequence

## How to apply

Load the OTU representative-sequence FASTA produced by the MICCA de novo OTU clustering step. Run the MICCA 'msa' command, which invokes MUSCLE (doi: 10.1093/nar/gkh340) to compute a fast, memory-efficient NAST multiple sequence alignment of the representative sequences. Then run the MICCA 'tree' command, which again uses MUSCLE for the alignment and FastTree (doi: 10.1371/journal.pone.0009490) to infer the phylogenetic tree. Because MICCA wraps these third-party tools, VSEARCH, MUSCLE, and FastTree must be cited when used. Validate that the output tree.nwk parses as Newick and contains exactly one leaf per input representative sequence.

## Related tools

- **MICCA** (Pipeline providing the 'msa' and 'tree' commands for alignment and phylogenetic inference) — https://github.com/compmetagen/micca
- **MUSCLE** (Underlying aligner invoked by both the micca 'msa' and 'tree' commands)
- **FastTree** (Phylogenetic tree inference engine invoked by the micca 'tree' command)

## Examples

```
micca msa -i otus_representatives.fasta -o otus_representatives_aligned.fasta; micca tree -i otus_representatives.fasta -t tree.nwk
```

## Evaluation signals

- tree.nwk exists and parses as a Newick-format tree without errors
- Number of leaves in the Newick tree equals the number of OTU representative sequences supplied as input
- The intermediate MSA covers the same sequence set as the input FASTA (no representatives silently dropped)
- Record which third-party tools (MUSCLE, FastTree, VSEARCH) and DOIs were used, since MICCA requires citing the wrapped software

## Limitations

- MICCA wraps third-party packages (MUSCLE, FastTree); correctness of the tree depends on those external tools being installed and cited
- The pipeline is designed for highly conserved genes/spacers such as 16S rRNA, ITS, 18S, and 28S rRNA amplicons — outside that scope alignment quality is not guaranteed
- Runs on Linux, Mac OS X, and MS Windows (through Docker containers); native execution on unsupported platforms may fail

## Evidence

- [abstract] efficiently combines quality filtering, clustering of Operational Taxonomic Units (OTUs), taxonomy assignment and phylogenetic tree inference: "Here we introduce MICCA, a software pipeline for the processing of amplicon metagenomic datasets that efficiently combines quality filtering, clustering of Operational Taxonomic Units (OTUs),"
- [readme] fast and and memory efficient **NAST** multiple sequence alignment (MSA): "fast and and memory efficient **NAST** multiple sequence alignment (MSA);"
- [readme] MUSCLE (doi: 10.1093/nar/gkh340) used in ``msa`` and ``tree`` commands: "MUSCLE (doi: 10.1093/nar/gkh340) used in ``msa`` and ``tree`` commands"
- [readme] FastTree (doi: 10.1371/journal.pone.0009490) used in the ``tree`` command: "FastTree (doi: 10.1371/journal.pone.0009490) used in the ``tree`` command"
