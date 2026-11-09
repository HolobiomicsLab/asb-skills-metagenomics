---
name: batch-file-workflow-orchestration
description: Use when you hold a set of per-locus FASTA files (one per schema locus,
  each containing all identified alleles) and need a per-locus MSA and neighbour-joining
  tree to assess locus-level allelic diversity.
when_to_use_negative:
- You only need a single whole-cgMLST MSA rather than per-locus alignments — the AlleleCallEvaluator/ComputeMSA
  module already produces the cgMLST MSA (protein_msa.fasta)
- Input files are genome assemblies rather than per-locus allele FASTAs; run CreateSchema/AlleleCall
  first to obtain the allelic diversity files
- The schema is an external schema not yet adapted via PrepExternalSchema, so locus
  FASTAs may not be in the expected chewBBACA format
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: chewBBACA
  role: Source of the per-locus allele FASTA files (full allelic diversity of each
    schema locus) that the batch pipeline iterates over
  repo: https://github.com/B-UMMI/chewBBACA
- name: MAFFT
  role: Builds the nucleotide multiple sequence alignment of all alleles for each
    locus FASTA in the batch
- name: ClustalW2
  role: Reconstructs a neighbour-joining phylogenetic tree from each per-locus MSA,
    one Newick file per locus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:5ab7914318c4287a1861945938378f3c7f3bf7b79996610b9895d83dcc1a2631
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# batch-file-workflow-orchestration

## Summary

Orchestrates a per-file batch pipeline over the per-locus allele FASTA files of a chewBBACA cg/wgMLST schema, running MAFFT to build one multiple sequence alignment per locus and ClustalW2 in neighbour-joining mode to build one tree per locus, so allelic variability at each locus can be explored.

## When to use

Use when you hold a set of per-locus FASTA files (one per schema locus, each containing all identified alleles) and need a per-locus MSA and neighbour-joining tree to assess locus-level allelic diversity.

## When NOT to use

- You only need a single whole-cgMLST MSA rather than per-locus alignments — the AlleleCallEvaluator/ComputeMSA module already produces the cgMLST MSA (protein_msa.fasta)
- Input files are genome assemblies rather than per-locus allele FASTAs; run CreateSchema/AlleleCall first to obtain the allelic diversity files
- The schema is an external schema not yet adapted via PrepExternalSchema, so locus FASTAs may not be in the expected chewBBACA format

## Inputs

- Per-locus FASTA files, one per schema locus, each containing the full allelic diversity of that locus (output of chewBBACA allele calling)

## Outputs

- One MAFFT nucleotide multiple sequence alignment file per locus (e.g. protein_msa.fasta-style MSA per locus)
- One ClustalW2 neighbour-joining Newick tree file per locus

## How to apply

Load the per-locus allele FASTA files that chewBBACA's allele call returns as 'a set of FASTA files containing the full allelic diversity of each locus'. For each locus FASTA, run MAFFT to produce a nucleotide multiple sequence alignment of all alleles, writing one MSA file per locus following the chewBBACA convention of an MSA file such as protein_msa.fasta in the evaluator output directory. For each per-locus MSA, run ClustalW2 in neighbour-joining mode to reconstruct a phylogenetic tree of the allele sequences as a Newick file. Before finishing, validate that every locus in the input set has both an MSA file and a tree file present in the output directory, so no locus is silently dropped from the batch.

## Related tools

- **chewBBACA** (Source of the per-locus allele FASTA files (full allelic diversity of each schema locus) that the batch pipeline iterates over) — https://github.com/B-UMMI/chewBBACA
- **MAFFT** (Builds the nucleotide multiple sequence alignment of all alleles for each locus FASTA in the batch)
- **ClustalW2** (Reconstructs a neighbour-joining phylogenetic tree from each per-locus MSA, one Newick file per locus)

## Evaluation signals

- Every locus in the input FASTA set has both an MSA file and a Newick tree file present in the output directory (one-to-one locus-to-output completeness check)
- Each per-locus MSA contains all alleles listed in that locus's input FASTA (no alleles dropped during alignment)
- Each tree file parses as valid Newick and its tips correspond to the alleles in the corresponding MSA
- MSA file naming follows the chewBBACA convention (e.g. protein_msa.fasta-style output in the evaluator directory)

## Limitations

- Output completeness was fragile in practice: the cgMLST MSA (protein_msa.fasta) went missing from AlleleCallEvaluator output between releases and was only re-added as a bug fix in 3.5.4, so output validation against the expected file set is essential
- Batch orchestration assumes each locus FASTA already contains the full allelic diversity; schemas altered in place by modules such as PrepExternalSchema (users are warned to backup data beforehand) may yield incomplete inputs

## Evidence

- [other] creating a multiple sequence alignment of each locus's alleles with MAFFT and reconstructing a neighbour-joining tree per locus using ClustalW2: [evidence span withheld — anchor status: not_found]
- [discussion] returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] The file with cgMLST MSA was re-added as one of the output files from the `AlleleCallEvaluator` module (`protein_msa.fasta` file).: "The file with cgMLST MSA was re-added as one of the output files from the `AlleleCallEvaluator` module (`protein_msa.fasta` file)."
- [discussion] be sure to backup your data before using this function: "be sure to backup your data before using this function"
