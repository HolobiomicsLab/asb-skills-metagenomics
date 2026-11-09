---
name: cgmlst-schema-locus-matching
description: Use when calling alleles of query-genome CDSs against a pre-defined cg/wgMLST
  schema and interpreting classification tags (EXC, INF, ASM, ALM, NIPH, NIPHEM, PLOT,
  LNF) in the allelic profile.
when_to_use_negative:
- Input is raw sequencing reads rather than assemblies — chewBBACA's allele-calling
  engine consumes FASTA assemblies; read-based callers (e.g. MentaLiST) are the alternative
- The task is schema construction or external-schema adaptation (CreateSchema/PrepExternalSchema),
  not allele calling against an existing schema
- The schema contains paralogous loci that were not screened — the tutorial advises
  removing the 20 RepeatedLoci.txt loci before core-genome extraction
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: chewBBACA
  role: Runs the AlleleCall BSR-based allele-calling engine and produces the allelic
    profile matrix
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlies the 100% identity exact-match screening and the BLASTp score-ratio
    classification of non-exact matches
- name: PrepExternalSchema
  role: Adapts schemas from external platforms (EnteroBase, PubMLST, Ridom) and removes
    non-translating alleles before allele calling
  repo: https://github.com/B-UMMI/chewBBACA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:2efd8e61aaef049d6fe714bb45da8482c0d14ca53236c89511e799cc1118f8c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - PrepExternalSchema
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cgmlst-schema-locus-matching

## Summary

Reconstructs chewBBACA's AlleleCall decision logic, which matches query-genome CDSs against schema loci and classifies them as exact matches (EXC), inferred alleles (INF), size-deviant ASM/ALM, paralogous NIPH/NIPHEM, contig-tip PLOT, or locus-not-found (LNF). It underpins gene-by-gene (cg/wgMLST) strain identification from draft assemblies.

## When to use

Use when calling alleles of query-genome CDSs against a pre-defined cg/wgMLST schema and interpreting classification tags (EXC, INF, ASM, ALM, NIPH, NIPHEM, PLOT, LNF) in the allelic profile.

## When NOT to use

- Input is raw sequencing reads rather than assemblies — chewBBACA's allele-calling engine consumes FASTA assemblies; read-based callers (e.g. MentaLiST) are the alternative
- The task is schema construction or external-schema adaptation (CreateSchema/PrepExternalSchema), not allele calling against an existing schema
- The schema contains paralogous loci that were not screened — the tutorial advises removing the 20 RepeatedLoci.txt loci before core-genome extraction

## Inputs

- Draft or complete genome FASTA assemblies
- Pre-defined cg/wgMLST schema (per-locus allele FASTA files, e.g. schema_seed directory or loci list passed to --gl)
- Optional Prodigal training file for the species (e.g. Streptococcus_agalactiae.trn)

## Outputs

- allelic_profile_matrix.tsv (integer allele identifiers or classification tags per genome-locus cell)
- Per-locus FASTA files of allelic diversity
- RepeatedLoci.txt listing paralogous loci
- Updated schema FASTAs including newly assigned novel allele identifiers

## How to apply

Run chewBBACA's AlleleCall with draft FASTA assemblies and the schema seed (loci FASTA files downloaded from Chewie-NS or adapted via PrepExternalSchema). CDS translations are BLASTed against a short-list of representative alleles per locus: hits at 100% nucleotide identity are exact matches (EXC), and remaining loci are resolved by a BLAST Score Ratio (BSR) approach against BLASTp matches — the default BSR threshold is 0.6 — assigning the allele number of the best representative or an inferred allele. Sequences whose length deviates from the locus mode (size threshold governs ASM/ALM) are tagged as alleles smaller than mode (ASM) or larger than mode (ALM). Loci matched in multiple distinct genomic regions are flagged as paralogous (NIPH/NIPHEM, listed in RepeatedLoci.txt and typically removed), matches at contig tips as PLOT, and loci with no valid hit as LNF. Outputs are compiled into an allelic profile matrix plus per-locus FASTA files of allelic diversity.

## Related tools

- **chewBBACA** (Runs the AlleleCall BSR-based allele-calling engine and produces the allelic profile matrix) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlies the 100% identity exact-match screening and the BLASTp score-ratio classification of non-exact matches)
- **PrepExternalSchema** (Adapts schemas from external platforms (EnteroBase, PubMLST, Ridom) and removes non-translating alleles before allele calling) — https://github.com/B-UMMI/chewBBACA

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Every genome-locus cell in allelic_profile_matrix.tsv contains either an integer allele identifier or a valid tag (EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF)
- Allele call with the default BSR threshold of 0.6; deviating thresholds should be justified against the wiki guidance
- Paralogous loci listed in RepeatedLoci.txt (e.g. 20 loci in the S. agalactiae wgMLST tutorial) are flagged and can be removed via RemoveGenes
- Novel alleles are added to the schema FASTAs (tutorial: schema grew from 3,128 to 17,848 alleles after calling 32 genomes)
- Per-locus sequence lengths conform to the locus mode except ASM/ALM-tagged deviations

## Limitations

- Paralogous loci create uncertainty in allele assignment and should be removed from the schema before downstream cgMLST extraction
- Size threshold governs ASM/ALM classification, so deviations from the locus mode are threshold-dependent
- Fixed bugs in select_highest_scores (sorting by sequence length instead of alignment raw score) could mis-classify some sequences in versions before 3.5.4
- Proprietary GbG platforms have incompletely described allele-calling algorithms, so classification conventions differ across tools

## Evidence

- [discussion] The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [discussion] Size threshold for ASM and ALM allele definition: "Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode)."
- [discussion] Output of an allelic profile matrix and FASTA files of allelic diversity per locus: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [readme] Default BSR threshold of 0.6: [evidence span withheld — anchor status: external_doc]
- [readme] Paralogous loci list: [evidence span withheld — anchor status: external_doc]
