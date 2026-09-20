---
name: blastp-all-against-all-sequence-clustering
description: Use when building a novel wg/cgMLST schema seed from a set of draft or
  complete genome assemblies and loci must be defined de novo by sequence-similarity
  clustering of predicted CDSs.
when_to_use_negative:
- Input consists of raw reads rather than assemblies — chewBBACA's CreateSchema and
  allele calling operate on FASTA draft assemblies or complete genomes
- An existing curated schema is already available (e.g. from Chewie-NS, EnteroBase,
  PubMLST, or Ridom cgMLST) — use PrepExternalSchema adaptation instead of de novo
  CreateSchema
- The goal is only allele calling against a finished schema, not schema construction
  — run AlleleCall directly
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: chewBBACA
  role: Suite providing the CreateSchema module that implements the BSR-based all-against-all
    BLASTP clustering of CDSs into loci
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Performs the all-against-all BLASTP similarity search; highest scoring matches
    per target are selected via select_highest_scores
- name: Prodigal
  role: Predicts CDSs on each input draft or complete genome assembly as the first
    CreateSchema step; species training files ship with chewBBACA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:61a6b48ed43dbf567d1fc07bf466be4ad719ffbb2438620b529f2d2b7226b4eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Prodigal
  - chewBBACA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# blastp-all-against-all-sequence-clustering

## Summary

Schema-construction strategy used by chewBBACA's CreateSchema: predicted CDSs from input genomes are clustered into wg/cgMLST loci via an all-against-all BLASTP comparison, merging sequences with BLAST Score Ratio (BSR) >= 0.6 as alleles of the same locus and retaining the largest allele as representative.

## When to use

Use when building a novel wg/cgMLST schema seed from a set of draft or complete genome assemblies and loci must be defined de novo by sequence-similarity clustering of predicted CDSs.

## When NOT to use

- Input consists of raw reads rather than assemblies — chewBBACA's CreateSchema and allele calling operate on FASTA draft assemblies or complete genomes
- An existing curated schema is already available (e.g. from Chewie-NS, EnteroBase, PubMLST, or Ridom cgMLST) — use PrepExternalSchema adaptation instead of de novo CreateSchema
- The goal is only allele calling against a finished schema, not schema construction — run AlleleCall directly

## Inputs

- Draft or complete genome assemblies as FASTA files (e.g. a directory like complete_genomes/)
- Optional Prodigal training file for the species of interest (e.g. .trn file)
- Optionally, a single multi-FASTA file of representative CDSs to skip gene prediction

## Outputs

- Schema seed directory (schema_seed) with one FASTA file per locus containing the locus's representative allele
- Set of loci each represented by a single sequence, ready for AlleleCall

## How to apply

Run Prodigal (with a species-specific training file, e.g. Streptococcus_agalactiae.trn) on each input assembly to predict CDSs. Filter the CDSs by removing duplicated shorter copies and any CDS below the minimum length threshold, then translate the remaining CDSs and perform an all-against-all BLASTP search. Cluster CDSs into loci by merging sequences whose BLAST Score Ratio is >= 0.6 (the default chewBBACA BSR threshold), keeping the larger allele as the locus representative, using BLAST score sorting (the select_highest_scores function) to select the best-scoring alignment per target. Write one FASTA file per locus under schema_seed containing the full allelic diversity; downstream AlleleCall uses the same 0.6 BSR threshold so loci and allele calls remain consistent. A schema can also be created from a single FASTA file, skipping gene prediction, when representative sequences are supplied directly.

## Related tools

- **chewBBACA** (Suite providing the CreateSchema module that implements the BSR-based all-against-all BLASTP clustering of CDSs into loci) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Performs the all-against-all BLASTP similarity search; highest scoring matches per target are selected via select_highest_scores)
- **Prodigal** (Predicts CDSs on each input draft or complete genome assembly as the first CreateSchema step; species training files ship with chewBBACA)

## Examples

```
chewBBACA.py CreateSchema -i complete_genomes/ -o tutorial_schema --ptf Streptococcus_agalactiae.trn --cpu 6
```

## Evaluation signals

- Every locus FASTA file in schema_seed exists and is valid FASTA, with each locus represented by a single sequence
- All retained alleles/loci meet the BSR >= 0.6 clustering criterion (no locus contains sequences with pairwise BSR below the threshold)
- No CDS below the minimum length threshold or duplicated shorter CDS copies remain in the schema seed
- Tutorial-scale expectation: 32 S. agalactiae complete genomes yield a wgMLST schema seed of 3,128 loci, and the first AlleleCall against it expands the schema to 17,848 alleles
- select_highest_scores sorting is based on alignment raw score (not target length), so the best-scoring alignment is chosen per target

## Limitations

- The BSR threshold of 0.6 is a fixed similarity criterion; sequences below it are treated as distinct loci, which may over-split highly divergent alleles
- Loci later flagged in RepeatedLoci.txt as possible paralogs (20 loci in the tutorial) should be removed from the schema because of potential allele-assignment uncertainty
- PrepExternalSchema (used when adapting external schemas via similar BLASTp steps) modifies the original source FASTA — back up data before using it
- A known bug (fixed in 3.5.4) sorted BLASTp matches by target length instead of alignment raw score, which could miss the best-scoring alignment and slightly reduce accuracy

## Evidence

- [methods] CreateSchema defines CDSs with Prodigal, removes shorter duplicate CDSs and CDSs below a minimum length, then clusters remaining CDSs via all-against-all blastp: CDSs with BSR >= 0.6 are alleles of the same locus: [evidence span withheld — anchor status: not_found]
- [readme] The 'BBACA' stands for 'BSR-Based Allele Calling Algorithm': "The "BBACA" stands for "BSR-Based Allele Calling Algorithm". BSR stands for BLAST Score Ratio as proposed by [Rasko DA et al.]"
- [intro] We developed chewBBACA to be a complete stand-alone pipeline for GbG analyses: "We developed chewBBACA to be a complete stand-alone pipeline for GbG analyses, including constructing and validating novel cg/wgMLST schemas"
- [methods] select_highest_scores function sorts and selects the highest scoring BLASTp matches per target: "the `select_highest_scores` function to sort and select the highest scoring BLASTp matches per target"
- [discussion] Schema creation can skip gene prediction when based on a single FASTA file: "New option for the schema creation. A schema can be created based on a single fasta file, skipping the gene prediction step."
- [readme] Tutorial command and result: 32 complete genomes yield a schema seed in tutorial_schema/schema_seed: [evidence span withheld — anchor status: external_doc]
