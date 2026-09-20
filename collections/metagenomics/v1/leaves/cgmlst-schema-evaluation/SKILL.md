---
name: cgmlst-schema-evaluation
description: Use when a chewBBACA schema (per-locus FASTA files with full allelic
  diversity) must be assessed locus-by-locus, e.g. to inspect allele variability before
  using the schema for cg/wgMLST strain identification.
when_to_use_negative:
- Input is raw sequencing reads or draft assemblies without a schema — perform allele
  calling first (chewBBACA AlleleCall), not per-locus MSA/tree evaluation
- Only an allelic profile matrix is needed (e.g. core-genome extraction with ExtractCgMLST
  or paralog removal with RemoveGenes) — this skill builds per-locus alignments and
  trees, which is unnecessary
- The schema loci are known paralogs listed in RepeatedLoci.txt — such loci carry
  uncertainty in allele assignment and should be removed rather than evaluated for
  phylogenetic structure
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: chewBBACA
  role: Suite providing schema creation/evaluation modules whose outputs (per-locus
    allele FASTAs, evaluator output directory) are the subject of this skill
  repo: https://github.com/B-UMMI/chewBBACA
- name: MAFFT
  role: Builds the nucleotide multiple sequence alignment of all alleles for each
    schema locus
- name: ClustalW2
  role: Reconstructs a neighbour-joining phylogenetic tree of the allele sequences
    from each per-locus MSA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:afa1c433b54784792c1093bbd2266b877d6fd89102d03be738c3ee6146ad1737
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
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cgmlst-schema-evaluation

## Summary

Per-locus evaluation of a cg/wgMLST schema by reconstructing, for each locus, a MAFFT multiple sequence alignment of all identified alleles and a ClustalW2 neighbour-joining tree, enabling exploration of the consequences of allelic variability at each locus in the chewBBACA SchemaEvaluator workflow.

## When to use

Use when a chewBBACA schema (per-locus FASTA files with full allelic diversity) must be assessed locus-by-locus, e.g. to inspect allele variability before using the schema for cg/wgMLST strain identification.

## When NOT to use

- Input is raw sequencing reads or draft assemblies without a schema — perform allele calling first (chewBBACA AlleleCall), not per-locus MSA/tree evaluation
- Only an allelic profile matrix is needed (e.g. core-genome extraction with ExtractCgMLST or paralog removal with RemoveGenes) — this skill builds per-locus alignments and trees, which is unnecessary
- The schema loci are known paralogs listed in RepeatedLoci.txt — such loci carry uncertainty in allele assignment and should be removed rather than evaluated for phylogenetic structure

## Inputs

- Per-locus FASTA files (one file per schema locus, containing all identified alleles for that locus)
- chewBBACA schema directory (e.g. schema_seed) produced by CreateSchema/AlleleCall

## Outputs

- One MAFFT multiple sequence alignment file per locus (e.g. protein_msa.fasta convention in the evaluator output directory)
- One ClustalW2 neighbour-joining Newick tree file per locus

## How to apply

Load the per-locus allele FASTA files produced by the allele-calling engine, which 'returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus'. For each locus FASTA, run MAFFT to build a nucleotide multiple sequence alignment of all alleles, writing one MSA file per locus in the evaluator output directory (following the chewBBACA convention, e.g. a protein_msa.fasta file). For each locus MSA, run ClustalW2 in neighbour-joining mode to reconstruct a per-locus Newick tree of the allele sequences. Before finishing, validate that every input locus has both an MSA file and a tree file present in the output directory, mirroring the chewBBACA practice of validating which steps ran in the evaluator output directory.

## Related tools

- **chewBBACA** (Suite providing schema creation/evaluation modules whose outputs (per-locus allele FASTAs, evaluator output directory) are the subject of this skill) — https://github.com/B-UMMI/chewBBACA
- **MAFFT** (Builds the nucleotide multiple sequence alignment of all alleles for each schema locus)
- **ClustalW2** (Reconstructs a neighbour-joining phylogenetic tree of the allele sequences from each per-locus MSA)

## Evaluation signals

- Every locus in the input set has BOTH an MSA file and a Newick tree file in the output directory before finishing
- Each per-locus MSA contains an alignment of all alleles present for that locus in the input FASTA (row count equals allele count)
- Each per-locus tree file parses as valid Newick and contains as many leaves as alleles in the locus FASTA
- The evaluator output directory includes the expected MSA artefacts (e.g. the protein_msa.fasta file, which was historically missing from the AlleleCallEvaluator output since the ComputeMSA module release)
- Alleles are translated/CDS-consistent — non-translating alleles are not silently retained, since chewBBACA enforces that alleles that do not translate are removed from the FASTA

## Limitations

- Paralogous loci (reported in RepeatedLoci.txt by the AlleleCall) introduce uncertainty in allele assignment and should be removed from the schema before interpreting per-locus trees
- The BSR allele-calling threshold (default 0.6) and size threshold (ASM/ALM alleles smaller/larger than mode) determine which alleles enter the per-locus FASTAs, so tree/MSA results depend on allele-calling parameters
- Execution time depends on hardware; the tutorial reports times for a DELL XPS13 (i7, 6 cores) and warns that less powerful machines greatly increase analysis duration

## Evidence

- [abstract] in the creation and evaluation of novel whole-genome or core-genome gene-by-gene typing schemas and subsequent allele calling in bacterial strains of interest: "in the creation and evaluation of novel whole-genome or core-genome gene-by-gene typing schemas and subsequent allele calling in bacterial strains of interest"
- [discussion] returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] The file with the cgMLST MSA was missing from the output directory created by the `AlleleCallEvaluator` module since the release of the `ComputeMSA` module: "The file with the cgMLST MSA was missing from the output directory created by the `AlleleCallEvaluator` module since the release of the `ComputeMSA` module"
- [readme] These loci should be removed from the schema due to the potential uncertainty in allele assignment: [evidence span withheld — anchor status: external_doc]
