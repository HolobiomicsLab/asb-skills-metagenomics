---
name: cds-deduplication-and-length-filtering
description: Use when constructing a wg/cgMLST schema from draft or complete genome
  assemblies and you need to clean Prodigal-predicted CDSs prior to all-against-all
  BLASTP clustering, or when adapting/validating external schemas that must contain
  only translating, non-redundant alleles.
when_to_use_negative:
- You only need to perform allele calling against an existing validated schema — use
  AlleleCall, which also has ASM/ALM size-threshold handling but does not re-filter
  schema construction input
- Input is raw sequencing reads rather than assembled genomes — schema creation in
  chewBBACA operates on FASTA draft assemblies or complete genomes
- You need to preserve all duplicate copies (e.g. for copy-number analysis) — this
  step explicitly removes duplicated shorter copies of identical CDSs
tools:
- name: chewBBACA
  role: CreateSchema module performs the CDS deduplication and minimum-length filtering
    during schema construction; PrepExternalSchema enforces removal of non-translating
    alleles during schema adaptation
  repo: github.com/B-UMMI/chewBBACA
- name: Prodigal
  role: Predicts CDSs on each input draft or complete genome assembly, producing the
    translated sequences that are then deduplicated and length-filtered
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:501def44676fd86eb6eb4b4b12fd39e7bc94e13ee4e83340bda52b398a6e457a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Prodigal
  - chewBBACA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# CDS deduplication and length filtering

## Summary

A preprocessing step in chewBBACA's CreateSchema that removes duplicated shorter copies of identical predicted CDSs and discards CDSs below a minimum length threshold before schema loci are defined by all-against-all blastp clustering. It guarantees that only unique, size-compliant coding sequences enter BSR-based locus clustering.

## When to use

Use when constructing a wg/cgMLST schema from draft or complete genome assemblies and you need to clean Prodigal-predicted CDSs prior to all-against-all BLASTP clustering, or when adapting/validating external schemas that must contain only translating, non-redundant alleles.

## When NOT to use

- You only need to perform allele calling against an existing validated schema — use AlleleCall, which also has ASM/ALM size-threshold handling but does not re-filter schema construction input
- Input is raw sequencing reads rather than assembled genomes — schema creation in chewBBACA operates on FASTA draft assemblies or complete genomes
- You need to preserve all duplicate copies (e.g. for copy-number analysis) — this step explicitly removes duplicated shorter copies of identical CDSs

## Inputs

- Draft or complete genome assemblies (FASTA) from de novo assemblers
- Prodigal-predicted and translated CDS sequences (FASTA protein sequences)
- Optional Prodigal training file (e.g. Streptococcus_agalactiae.trn)

## Outputs

- Deduplicated, minimum-length-filtered CDS set passed to all-against-all BLASTP clustering
- Schema seed loci (one FASTA file per locus under schema_seed/) containing only size-compliant, non-redundant alleles

## How to apply

Run Prodigal CDS prediction on each input draft or complete genome assembly (optionally supplying a species-specific Prodigal training file, e.g. Streptococcus_agalactiae.trn) to obtain translated coding sequences. Filter the predicted CDSs by removing duplicated shorter copies of identical CDSs, keeping one representative per identical sequence, and discard CDSs falling below the minimum length threshold. This reduces redundant all-against-all BLASTP comparisons and ensures that the downstream BSR-based clustering (BSR >= 0.6 merges alleles of the same locus, retaining the largest allele) operates on a clean, non-redundant set. When adapting external schemas (PrepExternalSchema), the same enforcement applies: alleles that do not translate are removed from the FASTA. Validation confirms every retained allele meets the size and translation criteria before locus FASTA files are written to the schema_seed directory.

## Related tools

- **chewBBACA** (CreateSchema module performs the CDS deduplication and minimum-length filtering during schema construction; PrepExternalSchema enforces removal of non-translating alleles during schema adaptation) — github.com/B-UMMI/chewBBACA
- **Prodigal** (Predicts CDSs on each input draft or complete genome assembly, producing the translated sequences that are then deduplicated and length-filtered)

## Examples

```
chewBBACA.py CreateSchema -i complete_genomes/ -o tutorial_schema --ptf Streptococcus_agalactiae.trn --cpu 6
```

## Evaluation signals

- No two identical CDS copies of differing length remain in the filtered set — shorter duplicates are removed
- All retained CDSs meet the minimum length threshold used by CreateSchema
- During schema adaptation, all retained alleles translate (non-translating alleles are removed from the FASTA)
- Downstream clustering invariant: after filtering, CDSs merged into one locus all satisfy BSR >= 0.6 with the largest allele retained as representative
- Schema seed directory contains exactly one valid FASTA per locus (e.g. 3128 loci for the 32-genome S. agalactiae tutorial)

## Limitations

- The minimum length and size-threshold parameters directly affect ASM and ALM allele definitions (alleles smaller/larger than mode), so misconfiguration propagates to allele calling
- PrepExternalSchema modifies the original source FASTA, requiring users to back up data beforehand
- Filtering is a preprocessing heuristic — paralogous loci with similar sequences are not resolved here and must be detected later (e.g. via RepeatedLoci.txt from AlleleCall)

## Evidence

- [full_text] CreateSchema defines CDSs with Prodigal, removes shorter duplicate CDSs and CDSs below a minimum length, then clusters remaining CDSs via all-against-all blastp: [evidence span withheld — anchor status: not_found]
- [full_text] Filter the predicted CDSs by removing duplicated shorter copies of identical CDSs and discarding CDSs below the minimum length threshold.: [evidence span withheld — anchor status: not_found]
- [discussion] It is now enforced that the alleles that do not translate are removed from the fasta: "It is now enforced that the alleles that do not translate are removed from the fasta"
- [discussion] Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode).: "Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode)."
- [readme] The schema seed will be available at `tutorial_schema/schema_seed`.: [evidence span withheld — anchor status: external_doc]
