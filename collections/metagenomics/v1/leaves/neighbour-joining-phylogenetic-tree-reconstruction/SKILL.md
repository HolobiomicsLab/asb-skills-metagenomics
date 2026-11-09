---
name: neighbour-joining-phylogenetic-tree-reconstruction
description: Use when you have per-locus allele FASTA files from a chewBBACA schema
  and want per-locus MSAs and neighbour-joining trees to assess allelic variability
  within individual schema loci.
when_to_use_negative:
- Input is a single multi-locus allelic profile matrix rather than per-locus allele
  FASTA files — the skill requires one FASTA per locus with full allelic diversity
- The goal is strain-level phylogeny or core-genome phylogenetic reconstruction across
  genomes rather than per-locus assessment of allelic variability within a schema
- The schema has not been allele called yet and per-locus allele FASTA files do not
  exist
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: chewBBACA
  role: SchemaEvaluator operation that orchestrates per-locus MSA and tree construction
    from allele FASTA files of a cg/wgMLST schema
  repo: https://github.com/B-UMMI/chewBBACA
- name: MAFFT
  role: Builds the nucleotide multiple sequence alignment of all alleles for each
    locus FASTA
- name: ClustalW2
  role: Reconstructs a neighbour-joining phylogenetic tree from each per-locus MSA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:66e3eccd28ce64274b2775e89bab41f392cbfe09d0d4867bffb19da0e6dd7514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - ClustalW2
  - MAFFT
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# neighbour-joining-phylogenetic-tree-reconstruction

## Summary

Within the chewBBACA SchemaEvaluator operation, each per-locus FASTA file containing the full allelic diversity of a cg/wgMLST schema locus is aligned with MAFFT and a neighbour-joining tree of its alleles is reconstructed with ClustalW2, enabling exploration of the potential consequences of allelic variability at each locus.

## When to use

Use when you have per-locus allele FASTA files from a chewBBACA schema and want per-locus MSAs and neighbour-joining trees to assess allelic variability within individual schema loci.

## When NOT to use

- Input is a single multi-locus allelic profile matrix rather than per-locus allele FASTA files — the skill requires one FASTA per locus with full allelic diversity
- The goal is strain-level phylogeny or core-genome phylogenetic reconstruction across genomes rather than per-locus assessment of allelic variability within a schema
- The schema has not been allele called yet and per-locus allele FASTA files do not exist

## Inputs

- Per-locus FASTA files (one per schema locus, each containing all identified alleles) chained from chewBBACA allele calling output

## Outputs

- One nucleotide MAFFT multiple sequence alignment file per locus (e.g. a protein_msa.fasta-style MSA in the evaluator output directory)
- One Newick neighbour-joining phylogenetic tree file per locus, reconstructed from the locus MSA with ClustalW2

## How to apply

Start from the chained per-locus allele FASTA files produced by chewBBACA allele calling, which contain the full allelic diversity of each locus (chewBBACA 'returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus'). For each locus FASTA, run MAFFT to build a nucleotide multiple sequence alignment of all identified alleles at that locus, writing one MSA file per locus in the evaluator output directory (following the chewBBACA convention of an MSA file such as protein_msa.fasta). For each per-locus MSA, run ClustalW2 in neighbour-joining mode to reconstruct a phylogenetic tree of the allele sequences and write one Newick tree file per locus. Before finishing, validate that every locus in the input set has both an MSA file and a tree file present in the output directory, since this per-locus MSA/tree pairing is what lets you explore the potential consequences of allelic variability at each locus.

## Related tools

- **chewBBACA** (SchemaEvaluator operation that orchestrates per-locus MSA and tree construction from allele FASTA files of a cg/wgMLST schema) — https://github.com/B-UMMI/chewBBACA
- **MAFFT** (Builds the nucleotide multiple sequence alignment of all alleles for each locus FASTA)
- **ClustalW2** (Reconstructs a neighbour-joining phylogenetic tree from each per-locus MSA)

## Evaluation signals

- Every locus in the input set has both an MSA file and a tree file present in the output directory before the workflow finishes
- Each MSA is a nucleotide multiple sequence alignment of all alleles at that locus, produced by MAFFT from the locus FASTA
- Each tree file is a Newick-format neighbour-joining tree reconstructed by ClustalW2 from the corresponding locus MSA
- The number of per-locus MSA files and per-locus tree files matches the number of input locus FASTA files
- MSA files follow the chewBBACA evaluator output convention (e.g. a protein_msa.fasta file in the evaluator output directory)

## Limitations

- The workflow explicitly requires a validation step confirming both MSA and tree files exist for every locus before finishing; a missing output for any locus is a failure state
- The per-locus trees are neighbour-joining trees of alleles at a single locus and are intended to explore the consequences of allelic variability, not to serve as genome- or strain-level phylogenies
- chewBBACA output conventions matter: e.g. the protein_msa.fasta file was unintentionally missing from AlleleCallEvaluator output after the ComputeMSA release and was re-added in chewBBACA 3.5.4, so older releases may lack expected MSA outputs

## Evidence

- [full_text] Per-locus MSA/tree pipeline from source card: [evidence span withheld — anchor status: not_found]
- [full_text] Per-locus tree reconstruction: [evidence span withheld — anchor status: not_found]
- [full_text] Output validation requirement: [evidence span withheld — anchor status: not_found]
- [discussion] Per-locus allele FASTA provenance: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] Evaluator MSA output convention and fix: "The file with cgMLST MSA was re-added as one of the output files from the `AlleleCallEvaluator` module (`protein_msa.fasta` file)."
