---
name: allelic-profile-matrix-generation
description: Use when draft/complete bacterial genome FASTA assemblies must be converted
  into a gene-by-gene allelic profile matrix against a pre-defined cgMLST/wgMLST schema,
  e.g. for strain typing or outbreak analysis.
when_to_use_negative:
- Input is raw sequencing reads rather than de novo assembled genomes — the AlleleCall
  engine uses FASTA files with draft assemblies or complete genomes as input
- The goal is schema creation rather than allele calling against an existing schema
  (use CreateSchema first); AlleleCall relies on existing schema loci and allele definitions
- A proprietary web platform's allele-calling algorithm is required for exact replication
  — commercial GbG platforms such as SeqSphere+ and Bionumerics are closed-source
  and incompletely described
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0194
tools:
- name: chewBBACA
  role: Runs the AlleleCall module implementing the BSR-based two-tier allele calling
    that produces the allelic profile matrix
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlying sequence search used for exact-match nucleotide comparison and
    BLASTp score-ratio classification of inferred alleles
- name: chewBBACA_tutorial
  role: Step-by-step worked example (714 Streptococcus agalactiae genomes) demonstrating
    AlleleCall usage, RepeatedLoci removal, and ExtractCgMLST
  repo: https://github.com/B-UMMI/chewBBACA_tutorial
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:3658195cc487ad16ff3e43aee617f907ba545c3cd4af870557d3c0c448da28ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - chewBBACA
  - chewBBACA_tutorial
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# allelic-profile-matrix-generation

## Summary

Generates an allelic profile matrix by calling CDS alleles of query genomes against a cg/wgMLST schema using chewBBACA's BSR-based AlleleCall engine, classifying hits as EXC, INF, ASM, ALM, NIPH/NIPHEM, PLOT, or LNF.

## When to use

Use when draft/complete bacterial genome FASTA assemblies must be converted into a gene-by-gene allelic profile matrix against a pre-defined cgMLST/wgMLST schema, e.g. for strain typing or outbreak analysis.

## When NOT to use

- Input is raw sequencing reads rather than de novo assembled genomes — the AlleleCall engine uses FASTA files with draft assemblies or complete genomes as input
- The goal is schema creation rather than allele calling against an existing schema (use CreateSchema first); AlleleCall relies on existing schema loci and allele definitions
- A proprietary web platform's allele-calling algorithm is required for exact replication — commercial GbG platforms such as SeqSphere+ and Bionumerics are closed-source and incompletely described

## Inputs

- Draft or complete genome assemblies in FASTA format
- Pre-defined cg/wgMLST schema (schema_seed FASTA loci, e.g. from Chewie-NS, PubMLST, EnteroBase, or Ridom via PrepExternalSchema)
- Optional Prodigal training file (e.g. Streptococcus_agalactiae.trn)
- Optional --gl list of core loci to restrict calling (e.g. cgMLSTschema.txt)

## Outputs

- Allelic profile matrix (results_alleles.tsv / allelic_profile_matrix.tsv)
- Per-locus FASTA files containing the full allelic diversity of each locus
- RepeatedLoci.txt list of paralogous loci
- Updated schema with newly assigned novel allele identifiers

## How to apply

Load draft genome assemblies and the schema seed FASTA loci (from Chewie-NS or adapted via PrepExternalSchema), then run chewBBACA's AlleleCall with a chosen BSR threshold (default 0.6). CDSs with 100% nucleotide identity to schema alleles are exact matches (EXC); non-exact hits are classified via BLASTp blast score ratio against a short-list of representative alleles, and novel alleles receive new integer identifiers and are added to the schema. Sequences deviating more than 20% from the locus mode length are tagged ASM (smaller) or ALM (larger); multiple distinct genomic hits are NIPH/NIPHEM (paralogous), contig-tip matches are PLOT, and loci without a valid match are LNF. The per-genome calls are compiled into an allelic profile matrix (results_alleles.tsv / allelic_profile_matrix.tsv) plus per-locus FASTA files of allelic diversity. Paralogous loci listed in RepeatedLoci.txt should be removed with RemoveGenes before downstream cgMLST analysis.

## Related tools

- **chewBBACA** (Runs the AlleleCall module implementing the BSR-based two-tier allele calling that produces the allelic profile matrix) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying sequence search used for exact-match nucleotide comparison and BLASTp score-ratio classification of inferred alleles)
- **chewBBACA_tutorial** (Step-by-step worked example (714 Streptococcus agalactiae genomes) demonstrating AlleleCall usage, RepeatedLoci removal, and ExtractCgMLST) — https://github.com/B-UMMI/chewBBACA_tutorial

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Every genome-locus cell in the allelic profile matrix contains either a valid integer allele identifier or one of the classification tags (EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF)
- Allele calling with default settings uses a BSR threshold of 0.6 — deviations should be justified
- Novel alleles found during calling are added to the schema, so schema allele counts should increase (tutorial: 3,128 to 17,848 alleles after calling 32 genomes)
- RepeatedLoci.txt paralogous loci should be removed with RemoveGenes before computing the core genome (tutorial: 20 loci removed, leaving 3,108 loci)
- Output directory contains an allelic profile matrix plus per-locus FASTA files of allelic diversity, matching the chewBBACA output convention

## Limitations

- Paralogous loci (NIPH/NIPHEM, listed in RepeatedLoci.txt) carry potential uncertainty in allele assignment and should be removed from the schema
- Loci may be missed due to sequencing coverage and assembly problems, motivating a 95% core-genome threshold rather than strict 100% presence
- PrepExternalSchema modifies the original source FASTA — back up data before use
- BLASTp best-match selection depended on a sorting index bug (issue #234) that could misclassify some sequences in older versions; use a fixed release

## Evidence

- [readme] The allele call used the default BSR threshold of 0.6 and took approximately 17 minutes to complete (an average of 32 seconds per genome).: [evidence span withheld — anchor status: external_doc]
- [readme] The allele call identified 14,720 novel alleles and added those alleles to the schema, increasing the number of alleles in the schema from 3,128 to 17,848.: [evidence span withheld — anchor status: external_doc]
- [discussion] Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode).: "Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode)."
- [discussion] The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input and returns an allelic profile matrix.: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [readme] The results were being sorted based on the length of the target sequences instead of the alignment raw score, potentially leading to some sequences not being classified.: "The results were being sorted based on the length of the target sequences instead of the alignment raw score."
