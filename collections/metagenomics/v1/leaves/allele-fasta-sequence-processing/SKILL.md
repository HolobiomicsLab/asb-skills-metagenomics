---
name: allele-fasta-sequence-processing
description: Use when you have one FASTA file per locus with all identified alleles
  from a chewBBACA cg/wgMLST schema and need to assess allelic variability per locus
  via per-locus MSAs and phylogenetic trees.
when_to_use_negative:
- Input is genome assemblies or reads rather than per-locus allele FASTA files — use
  chewBBACA allele calling (AlleleCall) instead.
- You need a single whole-cgMLST alignment rather than per-locus assessments — use
  the ComputeMSA module, which produces the cgMLST MSA (protein_msa.fasta) via the
  AlleleCallEvaluator.
- The schema loci have not yet been filtered for paralogs (e.g. the 20 RepeatedLoci.txt
  loci in the tutorial), since paralogous loci confound per-locus variability interpretation.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0194
tools:
- name: chewBBACA
  role: Source of the per-locus allele FASTA files (allele calling returns FASTA files
    with the full allelic diversity of each locus); SchemaEvaluator operation wraps
    the per-locus MSA and tree reconstruction
  repo: https://github.com/B-UMMI/chewBBACA
- name: MAFFT
  role: Builds a nucleotide multiple sequence alignment of all alleles at each locus,
    one MSA per locus
- name: ClustalW2
  role: Reconstructs a neighbour-joining phylogenetic tree from each locus MSA, one
    Newick tree per locus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:c9613a88921e8e42c5c3b6c9e356fb0ba7fe9e43f4d83210cc32ec3908359312
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - ClustalW2
  - MAFFT
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# allele-fasta-sequence-processing

## Summary

Processes the per-locus allele FASTA files output by chewBBACA (each containing the full allelic diversity of a locus) by building a MAFFT nucleotide multiple sequence alignment per locus and reconstructing a ClustalW2 neighbour-joining tree per locus, enabling exploration of the consequences of allelic variability at each schema locus.

## When to use

Use when you have one FASTA file per locus with all identified alleles from a chewBBACA cg/wgMLST schema and need to assess allelic variability per locus via per-locus MSAs and phylogenetic trees.

## When NOT to use

- Input is genome assemblies or reads rather than per-locus allele FASTA files — use chewBBACA allele calling (AlleleCall) instead.
- You need a single whole-cgMLST alignment rather than per-locus assessments — use the ComputeMSA module, which produces the cgMLST MSA (protein_msa.fasta) via the AlleleCallEvaluator.
- The schema loci have not yet been filtered for paralogs (e.g. the 20 RepeatedLoci.txt loci in the tutorial), since paralogous loci confound per-locus variability interpretation.

## Inputs

- One FASTA file per schema locus containing all identified alleles for that locus (chained from chewBBACA allele-calling output)

## Outputs

- One MAFFT nucleotide multiple sequence alignment file per locus (e.g. protein_msa.fasta-style output in the evaluator directory)
- One ClustalW2 neighbour-joining Newick tree file per locus

## How to apply

Load the per-locus allele FASTA files chained from the allele-calling output (chewBBACA returns 'a set of FASTA files containing the full allelic diversity of each locus'). For each locus FASTA, run MAFFT to produce a nucleotide multiple sequence alignment of all alleles at that locus, writing one MSA file per locus (following the chewBBACA convention of an MSA file such as protein_msa.fasta in the evaluator output directory). Then, for each locus MSA, run ClustalW2 in neighbour-joining mode to reconstruct a phylogenetic (Newick) tree of the allele sequences, writing one tree file per locus. Before finishing, validate that every locus in the input set has both an MSA file and a tree file present in the output directory. The rationale, per the chewBBACA SchemaEvaluator design, is that per-locus alignments and NJ trees let users explore the potential consequences of allelic variability at each locus before committing the schema to downstream analyses.

## Related tools

- **chewBBACA** (Source of the per-locus allele FASTA files (allele calling returns FASTA files with the full allelic diversity of each locus); SchemaEvaluator operation wraps the per-locus MSA and tree reconstruction) — https://github.com/B-UMMI/chewBBACA
- **MAFFT** (Builds a nucleotide multiple sequence alignment of all alleles at each locus, one MSA per locus)
- **ClustalW2** (Reconstructs a neighbour-joining phylogenetic tree from each locus MSA, one Newick tree per locus)

## Evaluation signals

- Every locus present in the input per-locus FASTA set has exactly one MSA file and one tree file in the output directory (paired MSA/tree completeness check).
- Each MAFFT MSA contains one sequence per input allele for that locus, so sequence count in the MSA matches the allele count in the source FASTA.
- Each tree file is a parseable Newick tree whose tips correspond to the alleles of that locus.
- MSA and tree files are located in the SchemaEvaluator output directory, following the chewBBACA convention of MSA files such as protein_msa.fasta.

## Limitations

- The cgMLST MSA (protein_msa.fasta) was at one point missing from the AlleleCallEvaluator output directory since the release of the ComputeMSA module (fixed in chewBBACA 3.5.4), so output inventories should be checked against the tool version.
- Interpretation assumes loci are non-paralogous; paralogous loci (e.g. the 20 RepeatedLoci.txt loci in the tutorial) create uncertainty in allele assignment and confound per-locus variability assessment.
- Allelic diversity depends on the allele-calling BSR threshold (default 0.6 in the tutorial); novel alleles added during allele calling (e.g. 14,720 novel alleles increasing the schema from 3,128 to 17,848 in the tutorial) change the per-locus FASTA contents.

## Evidence

- [methods] Per-locus MAFFT MSAs and ClustalW2 neighbour-joining trees: [evidence span withheld — anchor status: not_found]
- [discussion] Input: per-locus allele FASTAs: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] Allelic diversity changes via allele calling: [evidence span withheld — anchor status: external_doc]
- [readme] MSA file naming convention: "The file with cgMLST MSA was re-added as one of the output files from the `AlleleCallEvaluator` module (`protein_msa.fasta` file)."
