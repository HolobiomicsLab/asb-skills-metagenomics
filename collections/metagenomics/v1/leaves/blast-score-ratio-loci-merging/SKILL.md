---
name: blast-score-ratio-loci-merging
description: Use when constructing a novel wg/cgMLST schema seed from a set of draft
  or complete genome assemblies and you need to define distinct loci from predicted
  CDSs via BSR-based similarity clustering.
when_to_use_negative:
- Allele calling against an already-existing schema — use chewBBACA's AlleleCall instead
  of rebuilding loci from scratch
- Adapting an external schema from another cg/wgMLST platform (EnteroBase, PubMLST,
  Ridom) — use PrepExternalSchema
- Allele definitions must come directly from sequencing reads or an existing nomenclature
  rather than de novo assembly-derived CDSs (e.g. MentaLiST-style read-based calling
  is a different approach)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: chewBBACA
  role: Provides the CreateSchema module that implements CDS prediction, filtering,
    and BSR-based locus merging to build the schema seed
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: BLASTP all-against-all search on translated CDSs providing the pairwise alignment
    scores used to compute BSR for clustering
- name: Prodigal
  role: CDS prediction on each input draft or complete genome assembly prior to filtering
    and clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:6f3422bcabc7a78843a0f215f4e5c6670743401b8407f5ae3a0def3d041e1588
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Prodigal
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# blast-score-ratio-loci-merging

## Summary

Clustering step within chewBBACA's CreateSchema that merges translated CDSs into wg/cgMLST loci using an all-against-all BLASTP search: sequences with BLAST Score Ratio (BSR) >= 0.6 become alleles of the same locus, with the larger allele retained as representative.

## When to use

Use when constructing a novel wg/cgMLST schema seed from a set of draft or complete genome assemblies and you need to define distinct loci from predicted CDSs via BSR-based similarity clustering.

## When NOT to use

- Allele calling against an already-existing schema — use chewBBACA's AlleleCall instead of rebuilding loci from scratch
- Adapting an external schema from another cg/wgMLST platform (EnteroBase, PubMLST, Ridom) — use PrepExternalSchema
- Allele definitions must come directly from sequencing reads or an existing nomenclature rather than de novo assembly-derived CDSs (e.g. MentaLiST-style read-based calling is a different approach)

## Inputs

- Draft or complete bacterial genome assemblies in FASTA format (de novo assembled)
- Optional Prodigal training file for the target species (e.g. Streptococcus_agalactiae.trn)

## Outputs

- Schema seed directory containing one FASTA file per locus with the representative (largest) allele
- Set of loci where BSR >= 0.6 sequences have been merged into single loci

## How to apply

First run Prodigal CDS prediction on each input draft or complete genome assembly to obtain coding sequences, then filter out duplicated shorter copies of identical CDSs and discard CDSs below the minimum length threshold. Perform an all-against-all BLASTP search on the remaining translated CDSs to compute pairwise similarity, using the BLAST Score Ratio (Rasko et al.) rather than raw scores to normalize for allele length. Cluster CDSs into loci by merging sequences with BSR >= 0.6 (chewBBACA's default threshold), retaining the largest allele as the representative sequence for each locus. Write the resulting schema seed as one FASTA file per locus; downstream AlleleCall reuses the same 0.6 BSR threshold to assign known and novel alleles against this seed.

## Related tools

- **chewBBACA** (Provides the CreateSchema module that implements CDS prediction, filtering, and BSR-based locus merging to build the schema seed) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (BLASTP all-against-all search on translated CDSs providing the pairwise alignment scores used to compute BSR for clustering)
- **Prodigal** (CDS prediction on each input draft or complete genome assembly prior to filtering and clustering)

## Examples

```
chewBBACA.py CreateSchema -i complete_genomes/ -o tutorial_schema --ptf Streptococcus_agalactiae.trn --cpu 6
```

## Evaluation signals

- Every schema locus FASTA file in the schema_seed directory exists and is valid FASTA
- All retained alleles in a locus satisfy the BSR >= 0.6 clustering criterion
- Each schema locus contains a single representative (largest) allele at seed stage, e.g. the tutorial yielded 3,128 loci from 32 complete genomes
- Predicted CDSs below the minimum length threshold and duplicated shorter copies were removed before BLASTP
- Downstream AlleleCall against the seed (default BSR 0.6) assigns alleles without mass unclassified targets, confirming loci were properly merged

## Limitations

- Paralogous loci can arise and introduce uncertainty in allele assignment; the tutorial identified 20 loci as possible paralogs and recommends removing them from the schema
- Loci may be missed in some genomes due to sequencing coverage and assembly problems, which is why a 95% presence threshold is used to define the core genome
- Locally implemented bugs in BLAST match sorting (e.g. select_highest_scores sorting on length instead of alignment raw score) can cause best-scoring alignments to be missed; check the changelog/version

## Evidence

- [intro] BSR-based locus merging input/rationale: "chewBBACA performs the schema creation and allele calls on complete or draft genomes resulting from de novo assemblers"
- [readme] BSR definition: "BSR stands for BLAST Score Ratio as proposed by [Rasko DA et al.]"
- [readme] default BSR threshold: [evidence span withheld — anchor status: external_doc]
- [readme] schema seed output location: [evidence span withheld — anchor status: external_doc]
- [abstract] suite purpose for schema creation: "The chewBBACA suite was designed to assist users in the creation and evaluation of novel whole-genome or core-genome gene-by-gene typing schemas"
