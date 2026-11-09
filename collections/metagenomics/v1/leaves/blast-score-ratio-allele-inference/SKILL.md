---
name: blast-score-ratio-allele-inference
description: Use when allele calling draft/complete bacterial genome assemblies against
  a cg/wgMLST schema FASTA and each CDS must be classified as exact match, inferred
  (BSR-based), size-deviant (ASM/ALM), paralogous (NIPH), contig-tip (PLOT) or LNF.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled FASTA — chewBBACA's allele-calling
  engine takes FASTA files with draft assemblies or complete genomes as input
- The task is calling with an allele definition based on read mapping or k-mers (e.g.,
  MentaLiST-style) rather than BSR-based CDS classification from assemblies
- A schema has not been created or adapted — allele calling requires a schema seed;
  use schema-creation skills first
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: chewBBACA
  role: AlleleCall module performs the two-tier BSR-based classification of CDSs against
    schema loci and writes the allelic profile matrix
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: 'Underlying aligner: nucleotide exact-match detection and BLASTp hits against
    locus short-lists scored via blast score ratio'
- name: RemoveGenes
  role: Post-call filter that removes paralogous loci (RepeatedLoci.txt columns) from
    the allelic profiles
  repo: https://github.com/B-UMMI/chewBBACA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:c02e3e4f13125597ea986a97c997b0de3a715d323e0620c759c24e235efce253
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - RemoveGenes
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# blast-score-ratio-allele-inference

## Summary

Reproduces chewBBACA's AlleleCall decision logic, which classifies CDSs from draft assemblies against schema loci using exact-match DNA identity plus a BLAST Score Ratio (BSR) approach, emitting allele numbers or classification tags (EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF).

## When to use

Use when allele calling draft/complete bacterial genome assemblies against a cg/wgMLST schema FASTA and each CDS must be classified as exact match, inferred (BSR-based), size-deviant (ASM/ALM), paralogous (NIPH), contig-tip (PLOT) or LNF.

## When NOT to use

- Input is raw sequencing reads rather than assembled FASTA — chewBBACA's allele-calling engine takes FASTA files with draft assemblies or complete genomes as input
- The task is calling with an allele definition based on read mapping or k-mers (e.g., MentaLiST-style) rather than BSR-based CDS classification from assemblies
- A schema has not been created or adapted — allele calling requires a schema seed; use schema-creation skills first

## Inputs

- Draft or complete genome assemblies in FASTA format (e.g., the 32 S. agalactiae complete genomes or 682 NCBI assemblies)
- Pre-defined cg/wgMLST schema seed FASTA loci (from Chewie-NS, CreateSchema output, or adapted external schemas)
- Optional Prodigal training file (e.g., Streptococcus_agalactiae.trn)

## Outputs

- allelic_profile_matrix.tsv (results_alleles.tsv) with integer allele IDs or EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF tags per genome-locus cell
- RepeatedLoci.txt listing putative paralogous loci
- Per-locus FASTA files containing the full allelic diversity, including newly identified alleles added to the schema

## How to apply

Run chewBBACA's AlleleCall module with draft/complete FASTA assemblies and a schema seed; CDS translations are BLASTed against a short-list of representative alleles per locus, and hits at 100% nucleotide identity are exact matches (EXC). For loci without an exact match, BLASTp-based inference assigns an allele number when the blast score ratio supports it, with the default BSR threshold of 0.6 applied by AlleleCall. Inferred allele lengths are validated against the locus mode: sequences deviating more than 20% from the mode are tagged ASM (smaller) or ALM (larger). Hits of the same locus in multiple distinct genomic regions are tagged NIPH (NIPHEM when exact mid-hit matches accompany NIPH), matches at contig tips are tagged PLOT, and loci with no valid match are LNF; results are compiled into an allelic profile matrix. Paralogous loci reported in RepeatedLoci.txt should typically be pruned (e.g., via RemoveGenes) to avoid uncertain allele assignment.

## Related tools

- **chewBBACA** (AlleleCall module performs the two-tier BSR-based classification of CDSs against schema loci and writes the allelic profile matrix) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying aligner: nucleotide exact-match detection and BLASTp hits against locus short-lists scored via blast score ratio)
- **RemoveGenes** (Post-call filter that removes paralogous loci (RepeatedLoci.txt columns) from the allelic profiles) — https://github.com/B-UMMI/chewBBACA

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Every genome-locus cell in allelic_profile_matrix.tsv contains either a valid integer allele identifier or one of the tags EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF
- The default BSR threshold of 0.6 was used (or explicitly overridden) and all inferred classifications respect it
- ASM/ALM tags appear only for alleles whose size deviates more than 20% from the locus mode
- RepeatedLoci.txt is generated and paralogous loci flagged there (e.g., the tutorial's 20 loci) are removed before defining the cgMLST core
- Allele calling expands the schema as novel alleles are added (tutorial case: 3,128 to 17,848 alleles over 32 genomes)

## Limitations

- Paralogous loci (NIPH/NIPHEM) introduce uncertainty in allele assignment and should be removed from schemas; the tutorial flagged 20 such loci out of 3,128
- Size-threshold misclassification: the ASM/ALM determination depends on the locus mode and the 20% size threshold, so borderline alleles may be tagged rather than numbered
- The select_highest_scores BLASTp sorting bug fixed in chewBBACA 3.5.4 (sorted by length instead of alignment raw score) could leave some sequences unclassified in older versions
- Loci absent due to sequencing coverage and assembly problems may appear as LNF, which is why core-genome thresholds like 95% presence are used downstream

## Evidence

- [discussion] The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [discussion] returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [discussion] Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode).: "Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode)."
- [readme] The allele call used the default BSR threshold of 0.6: [evidence span withheld — anchor status: not_found]
- [readme] The RepeatedLoci.txt file contains a set of 20 loci that were identified as possible paralogs.: [evidence span withheld — anchor status: external_doc]
