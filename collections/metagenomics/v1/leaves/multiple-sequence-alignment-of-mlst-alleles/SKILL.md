---
name: multiple-sequence-alignment-of-mlst-alleles
description: Use when evaluating individual loci in a cg/wgMLST schema — i.e. when
  you have per-locus FASTA files of allelic diversity and want to inspect alignment/phylogenetic
  structure of alleles per locus.
when_to_use_negative:
- You only need an allelic profile matrix or allele calls, not per-locus alignment/phylogeny
  inspection — run chewBBACA AlleleCall instead
- You are adapting an external schema (PrepExternalSchema), which removes non-translating
  alleles rather than aligning allelic diversity
- Input is whole-cgMLST concatenated data rather than per-locus allele FASTA files
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MAFFT
  role: Builds the per-locus nucleotide multiple sequence alignment of all alleles
    at each schema locus
- name: ClustalW2
  role: Reconstructs a neighbour-joining phylogenetic tree per locus from the MAFFT
    alignment
- name: chewBBACA
  role: Provides the schema and per-locus allele FASTA inputs and the evaluator output
    directory structure for the alignments/trees
  repo: https://github.com/B-UMMI/chewBBACA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:18a0fda442a35e53d0a596e4aa0bd97aa25fd769b9d41f4c332367876fd841c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - ClustalW2
  - MAFFT
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# multiple-sequence-alignment-of-MLST-alleles

## Summary

Generates per-locus nucleotide multiple sequence alignments (MAFFT) and neighbour-joining trees (ClustalW2) for every locus in a chewBBACA cg/wgMLST schema, enabling assessment of the allelic variability at each locus during schema evaluation.

## When to use

Use when evaluating individual loci in a cg/wgMLST schema — i.e. when you have per-locus FASTA files of allelic diversity and want to inspect alignment/phylogenetic structure of alleles per locus.

## When NOT to use

- You only need an allelic profile matrix or allele calls, not per-locus alignment/phylogeny inspection — run chewBBACA AlleleCall instead
- You are adapting an external schema (PrepExternalSchema), which removes non-translating alleles rather than aligning allelic diversity
- Input is whole-cgMLST concatenated data rather than per-locus allele FASTA files

## Inputs

- Per-locus FASTA files of allelic diversity, one FASTA per schema locus with all identified alleles (chained from locus_allele_fastas)

## Outputs

- One MAFFT nucleotide multiple sequence alignment file per locus (e.g. protein_msa.fasta in the evaluator output directory)
- One ClustalW2 neighbour-joining Newick tree file per locus

## How to apply

Load the per-locus allele FASTA files output by chewBBACA allele calling (the 'set of FASTA files containing the full allelic diversity of each locus'). For each locus FASTA, run MAFFT to build a nucleotide multiple sequence alignment of all alleles at that locus, writing one MSA file per locus in the evaluator output directory (chewBBACA convention includes files such as protein_msa.fasta). Then, for each locus MSA, run ClustalW2 in neighbour-joining mode to reconstruct a phylogenetic tree of the allele sequences, writing one Newick tree file per locus. Validate before finishing that every locus in the input set has both an MSA file and a tree file present in the output directory; these per-locus alignments and trees let you explore the potential consequences of allelic variability at each locus.

## Related tools

- **MAFFT** (Builds the per-locus nucleotide multiple sequence alignment of all alleles at each schema locus)
- **ClustalW2** (Reconstructs a neighbour-joining phylogenetic tree per locus from the MAFFT alignment)
- **chewBBACA** (Provides the schema and per-locus allele FASTA inputs and the evaluator output directory structure for the alignments/trees) — https://github.com/B-UMMI/chewBBACA

## Evaluation signals

- Every locus in the input set has both an MSA file and a Newick tree file present in the output directory
- Each MSA covers all identified alleles at that locus (no allele sequences dropped)
- Tree files are valid Newick with one leaf per allele sequence in the corresponding MSA
- Output directory follows chewBBACA evaluator conventions, e.g. contains protein_msa.fasta among the output files

## Limitations

- Task-cited evaluation of allelic variability is exploratory — alignments/trees show potential consequences of allelic variability but do not validate allele nomenclature by themselves
- Paralogous loci (flagged in RepeatedLoci.txt by the allele call) carry potential uncertainty in allele assignment and may confound per-locus tree interpretation

## Evidence

- [discussion] FASTA files of allelic diversity per locus: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] MSA output convention: "The file with cgMLST MSA was re-added as one of the output files from the `AlleleCallEvaluator` module (`protein_msa.fasta` file)."
- [methods] evaluation module context: "the output directory created by the `AlleleCallEvaluator` module"
