---
name: cds-prediction-from-genome-assemblies
description: Use when the input is draft or complete genome assemblies (FASTA from
  de novo assemblers) and the goal is to define target loci for a novel wg/cgMLST
  schema from scratch, requiring CDS prediction rather than allele calling against
  an existing schema.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled FASTA genomes (chewBBACA operates
  on 'complete or draft genomes resulting from de novo assemblers')
- An existing wg/cgMLST schema from an external platform is to be used instead of
  defining novel loci — use PrepExternalSchema adaptation instead
- Only allele calling against an already-built schema seed is needed — use AlleleCall
  directly rather than CreateSchema
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: chewBBACA
  role: Suite providing the CreateSchema module that orchestrates CDS prediction,
    filtering, and BSR-based clustering into loci
  repo: https://github.com/B-UMMI/chewBBACA
- name: Prodigal
  role: Performs CDS prediction on each input draft or complete genome assembly, guided
    by a species training file
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: All-against-all BLASTP of translated, filtered CDSs to compute pairwise similarity
    for BSR >= 0.6 locus clustering
  repo: https://github.com/B-UMMI/chewBBACA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:90b064e62b723b644b57ab3efd9fa3d1cb7c4c6a67a9857c4055dd0ddc37ed44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Prodigal
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cds-prediction-from-genome-assemblies

## Summary

De novo prediction of coding sequences (CDSs) from draft or complete bacterial genome FASTA assemblies using Prodigal within chewBBACA's CreateSchema, as the first stage of wg/cgMLST schema construction; predicted CDSs are deduplicated, length-filtered, and clustered into loci by all-against-all BLASTP at BSR >= 0.6.

## When to use

Use when the input is draft or complete genome assemblies (FASTA from de novo assemblers) and the goal is to define target loci for a novel wg/cgMLST schema from scratch, requiring CDS prediction rather than allele calling against an existing schema.

## When NOT to use

- Input is raw sequencing reads rather than assembled FASTA genomes (chewBBACA operates on 'complete or draft genomes resulting from de novo assemblers')
- An existing wg/cgMLST schema from an external platform is to be used instead of defining novel loci — use PrepExternalSchema adaptation instead
- Only allele calling against an already-built schema seed is needed — use AlleleCall directly rather than CreateSchema

## Inputs

- Directory of draft and/or complete genome assembly FASTA files (de novo assembler output)
- Prodigal species-specific training file (.trn), e.g. Streptococcus_agalactiae.trn
- Number of CPU cores (--cpu)

## Outputs

- Schema seed directory (e.g. tutorial_schema/schema_seed) containing one FASTA file per locus with the representative/allelic sequences

## How to apply

Run chewBBACA CreateSchema on the directory of assemblies, supplying a species-matched Prodigal training file via --ptf (e.g. Streptococcus_agalactiae.trn, which ships with chewBBACA) to improve gene prediction accuracy. CreateSchema runs Prodigal CDS prediction on each input assembly, removes duplicated shorter copies of identical CDSs, and discards CDSs below the minimum length threshold. The remaining translated CDSs undergo an all-against-all BLASTP search; sequences with BLAST Score Ratio (BSR) >= 0.6 are merged as alleles of the same locus, retaining the largest allele as the representative. The result is written as a schema seed — a set of FASTA files, one per locus, each containing the allelic diversity. Note the decision point: if the schema is to be based on a single FASTA file, the gene prediction step can be skipped.

## Related tools

- **chewBBACA** (Suite providing the CreateSchema module that orchestrates CDS prediction, filtering, and BSR-based clustering into loci) — https://github.com/B-UMMI/chewBBACA
- **Prodigal** (Performs CDS prediction on each input draft or complete genome assembly, guided by a species training file) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (All-against-all BLASTP of translated, filtered CDSs to compute pairwise similarity for BSR >= 0.6 locus clustering) — https://github.com/B-UMMI/chewBBACA

## Examples

```
chewBBACA.py CreateSchema -i complete_genomes/ -o tutorial_schema --ptf Streptococcus_agalactiae.trn --cpu 6
```

## Evaluation signals

- The schema seed directory (e.g. tutorial_schema/schema_seed) exists and contains one FASTA file per locus, each valid FASTA
- In the tutorial benchmark (32 S. agalactiae complete genomes, 6 cores), CreateSchema completed in ~56 seconds and produced a wgMLST schema with 3128 loci — a useful sanity-check magnitude
- Every retained locus representative satisfies the BSR >= 0.6 clustering criterion; no locus representative was discarded by the minimum-length filter
- No duplicated shorter copies of identical CDSs remain among predicted CDSs after the deduplication filter
- A species-matched Prodigal training file (.trn) was passed via --ptf rather than relying on the default

## Limitations

- Allele calling and schema creation require assembled genomes (FASTA); raw reads must be assembled first
- Using an inappropriate or missing Prodigal training file can degrade CDS prediction quality for the species of interest
- CreateSchema based on a single FASTA file skips the gene prediction step, which changes the nature of the resulting schema
- The initial schema seed contains a single representative allele per locus; full allelic diversity accrues only after downstream AlleleCall (e.g. 3,128 loci grew to 17,848 alleles in the tutorial)

## Evidence

- [methods] CreateSchema defines CDSs with Prodigal, removes shorter duplicate CDSs and CDSs below a minimum length, then clusters remaining CDSs via all-against-all blastp: CDSs with BSR >= 0.6 are alleles of the same locus, keeping the larger allele.: [evidence span withheld — anchor status: not_found]
- [intro] chewBBACA performs the schema creation and allele calls on complete or draft genomes resulting from de novo assemblers: "chewBBACA performs the schema creation and allele calls on complete or draft genomes resulting from de novo assemblers"
- [readme] chewBBACA includes Prodigal training files for several species, including for _Streptococcus agalactiae_.: [evidence span withheld — anchor status: external_doc]
- [discussion] New option for the schema creation. A schema can be created based on a single fasta file, skipping the gene prediction step.: "A schema can be created based on a single fasta file, skipping the gene prediction step."
- [readme] In our system, the process took 56 seconds to complete resulting on a wgMLST schema with 3128 loci. At this point the schema is defined as a set of loci each with a single representative allele.: [evidence span withheld — anchor status: external_doc]
