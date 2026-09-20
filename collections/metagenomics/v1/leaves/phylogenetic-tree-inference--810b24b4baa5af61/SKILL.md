---
name: phylogenetic-tree-inference
description: Use when starting a PM-CNN analysis from an OTU/species feature table
  whose representative nucleotide sequences are available (e.g. from GreenGenes) and
  no phylogenetic tree in NEWICK format has yet been supplied.
when_to_use_negative:
- A phylogenetic tree in NEWICK format is already available (the README marks this
  step as optional and supports user-supplied NEWICK trees, e.g. for shotgun data)
- Only a species/OTU abundance table exists and the representative sequences cannot
  be retrieved from a reference database such as GreenGenes
- The input sequences are not nucleotide sequences (the -nt option configures FastTree
  for nucleotides)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0637
tools:
- name: Mafft
  role: Performs multiple sequence alignment (--auto) of representative OTU nucleotide
    sequences prior to tree inference
- name: FastTree
  role: Infers the phylogenetic tree from the aligned FASTA using -nt (nucleotide)
    and -gtr (generalized time-reversible) options, outputting NEWICK format
- name: PM-CNN
  role: Downstream framework whose cophenetic-distance step consumes the NEWICK tree
    produced by this skill
  repo: https://github.com/qdu-bioinfo/PM_CNN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:414efc1f495f30827506d3d802e02d90ec5d69a5f7d0c6e2383a16222e2f4394
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - FastTree
  - Mafft
  - PM-CNN
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# phylogenetic-tree-inference

## Summary

Constructs a phylogenetic tree from representative OTU marker-gene sequences by multiple sequence alignment with Mafft (--auto) followed by tree inference with FastTree (-nt -gtr), producing the NEWICK-format tree consumed by PM-CNN's cophenetic-distance step.

## When to use

Use when starting a PM-CNN analysis from an OTU/species feature table whose representative nucleotide sequences are available (e.g. from GreenGenes) and no phylogenetic tree in NEWICK format has yet been supplied.

## When NOT to use

- A phylogenetic tree in NEWICK format is already available (the README marks this step as optional and supports user-supplied NEWICK trees, e.g. for shotgun data)
- Only a species/OTU abundance table exists and the representative sequences cannot be retrieved from a reference database such as GreenGenes
- The input sequences are not nucleotide sequences (the -nt option configures FastTree for nucleotides)

## Inputs

- Representative nucleotide sequences of all OTUs (e.g. example_respresent_seqs.txt, marker gene or amplicon sequences retrieved from GreenGenes)
- Optionally, a pre-existing phylogenetic tree in NEWICK format supplied by the user (e.g. for shotgun data)

## Outputs

- Multiple sequence alignment in FASTA format (e.g. output_aligned.fasta, intermediate)
- Phylogenetic tree in NEWICK format (e.g. example.tree) whose tips correspond to the OTUs in the input dataset

## How to apply

First retrieve the representative nucleotide sequences of all OTUs from the GreenGenes database, since PM-CNN's tree must cover exactly the OTUs in the feature table. Align these sequences with 'mafft --auto' to yield an aligned FASTA file (e.g. output_aligned.fasta). Infer the phylogeny from the alignment with 'FastTree -nt -gtr', where -nt selects nucleotide mode and -gtr applies the generalized time-reversible substitution model, writing the result to a NEWICK file such as example.tree. This step is explicitly optional: if a NEWICK tree already exists (e.g. for shotgun data or a user-supplied phylogeny), skip straight to the cophenetic-distance step with the R package ape. Downstream, get_distance_matrix.R expects the NEWICK file produced here as its parameter2 input.

## Related tools

- **Mafft** (Performs multiple sequence alignment (--auto) of representative OTU nucleotide sequences prior to tree inference)
- **FastTree** (Infers the phylogenetic tree from the aligned FASTA using -nt (nucleotide) and -gtr (generalized time-reversible) options, outputting NEWICK format)
- **PM-CNN** (Downstream framework whose cophenetic-distance step consumes the NEWICK tree produced by this skill) — https://github.com/qdu-bioinfo/PM_CNN

## Examples

```
mafft --auto example/example_respresent_seqs.txt > example/output_aligned.fasta; FastTree -nt -gtr example/output_aligned.fasta > example/data/example.tree
```

## Evaluation signals

- The output file is syntactically valid NEWICK (parses as a tree, e.g. by ape's read.tree)
- Tree tips correspond exactly to the OTUs in the input dataset / feature table
- Mafft stage produces a non-empty aligned FASTA (output_aligned.fasta) covering all representative OTU sequences
- The NEWICK file is accepted by the next step, get_distance_matrix.R, which requires parameter2 to be a NEWICK tree file from FastTree
- Sequences were retrieved from GreenGenes so representative sequences match the OTU identifiers in the abundance table

## Limitations

- The step is optional and depends on external software (Mafft and FastTree) that must be downloaded separately; the README notes the related software can be downloaded from official websites or requested by e-mail
- Representative sequences must first be found from the GreenGenes database before alignment; sequences for non-GreenGenes features (e.g. some shotgun species) may not be recoverable, in which case a user-supplied NEWICK tree is required
- FastTree is configured with -gtr (generalized time-reversible); other substitution models are not exercised by the documented command

## Evidence

- [readme] Mafft alignment then FastTree commands: "mafft --auto example/example_respresent_seqs.txt > example/output_aligned.fasta // you need to download Mafft"
- [readme] FastTree command: "FastTree -nt -gtr example/output_aligned.fasta > example/data/example.tree  // you need to download FastTree"
- [readme] Optional step and NEWICK alternative: "This step is optional, the phylogeny tree can either be constructed from representative sequences (e.g. marker gene or amplicon), or provided by users from a NEWICK format file (e.g. for shotgun)."
- [readme] GreenGenes prerequisite: "Before performing multiple sequence alignment, the representative sequences of all OTUs need to be found from the GreenGenes database."
- [intro] Tree construction in article intro: "The evolutionary tree of the representative sequence is constructed by FastTree and Mafft"
- [readme] Downstream NEWICK consumption: "parameter2: Input a phylogenetic tree file in NEWICK format (This comes from the results generated by FastTree in step 1)"
