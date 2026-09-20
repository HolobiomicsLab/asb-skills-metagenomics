---
name: cds-allele-calling-classification
description: Use when draft or complete genome FASTA assemblies must be assigned allele
  identifiers or classification tags (EXC, INF, ASM, ALM, NIPH, NIPHEM, PLOT, LNF)
  against a pre-defined cg/wgMLST schema to build strain allelic profiles.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled FASTA genomes — AlleleCall operates
  on de novo-assembled complete or draft genomes, and read-based alternatives like
  MentaLiST exist for that case
- No schema exists yet — schema creation (CreateSchema) must precede allele calling;
  this skill assumes loci/alleles are already defined
- The external schema has not been adapted (e.g. EnteroBase, PubMLST, Ridom formats
  require PrepExternalSchema, which also removes non-translating alleles) before calling
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: chewBBACA
  role: Provides the AlleleCall module implementing the BSR-based allele-calling decision
    logic (EXC/INF, ASM/ALM, NIPH/NIPHEM, PLOT, LNF) and the RemoveGenes module to
    drop paralogous loci from the profile matrix
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: 'Underlying alignment engine: BLASTn-style exact-match detection (EXC) and
    BLASTp alignments whose raw scores feed the BSR-based classification of non-exact
    hits'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:8be5a39be834999a26d4997b44bec3ed42c56a83f077929a785dc6d0a1c7ea50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - chewBBACA
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cds-allele-calling-classification

## Summary

Reconstructs and applies chewBBACA's AlleleCall decision logic, which classifies query-genome CDSs against schema loci using BLAST-based BLAST Score Ratio (BSR) comparisons, tagging calls as exact matches (EXC/INF), size deviations (ASM/ALM), paralogous hits (NIPH/NIPHEM), contig-tip hits (PLOT), or locus-not-found (LNF). It is the core allele-calling step that turns draft genome assemblies into allelic profile matrices for cg/wgMLST analysis.

## When to use

Use when draft or complete genome FASTA assemblies must be assigned allele identifiers or classification tags (EXC, INF, ASM, ALM, NIPH, NIPHEM, PLOT, LNF) against a pre-defined cg/wgMLST schema to build strain allelic profiles.

## When NOT to use

- Input is raw sequencing reads rather than assembled FASTA genomes — AlleleCall operates on de novo-assembled complete or draft genomes, and read-based alternatives like MentaLiST exist for that case
- No schema exists yet — schema creation (CreateSchema) must precede allele calling; this skill assumes loci/alleles are already defined
- The external schema has not been adapted (e.g. EnteroBase, PubMLST, Ridom formats require PrepExternalSchema, which also removes non-translating alleles) before calling

## Inputs

- Draft or complete genome assemblies in FASTA format (e.g. complete_genomes/ directory)
- Pre-defined cg/wgMLST schema seed FASTA loci (e.g. tutorial_schema/schema_seed)
- Optional Prodigal training file for the species (e.g. Streptococcus_agalactiae.trn)
- Optional list of core loci to restrict calling (passed to --gl, e.g. cgMLSTschema.txt)

## Outputs

- Allelic profile matrix (results_alleles.tsv / allelic_profile_matrix.tsv) with integer allele IDs or classification tags per genome-locus cell
- Per-locus FASTA files containing the full allelic diversity of each locus
- RepeatedLoci.txt listing paralogous loci (NIPH/NIPHEM)
- Updated schema FASTA files incorporating newly assigned novel alleles

## How to apply

Load the draft genome assemblies and the schema seed FASTA loci (downloaded from Chewie-NS or created with CreateSchema), then run the AlleleCall module, which classifies CDSs showing 100% nucleotide identity to schema alleles as exact matches and resolves non-exact cases with a BLASTp BSR approach against a short-list of representative alleles, using the default BSR threshold of 0.6. Validate inferred allele lengths against the locus mode, tagging sequences deviating by more than the size threshold as ASM (smaller) or ALM (larger). Detect multiple distinct genomic regions matching one locus as paralogs (NIPH/NIPHEM) — AlleleCall reports these in RepeatedLoci.txt — flag matches at contig tips as PLOT, and assign LNF when no valid match is found. Novel alleles are assigned identifiers and added to the schema (in the tutorial, 14,720 novel alleles raised the schema from 3,128 to 17,848 alleles). Paralogous loci should subsequently be removed (e.g. via RemoveGenes) to avoid uncertain allele assignment.

## Related tools

- **chewBBACA** (Provides the AlleleCall module implementing the BSR-based allele-calling decision logic (EXC/INF, ASM/ALM, NIPH/NIPHEM, PLOT, LNF) and the RemoveGenes module to drop paralogous loci from the profile matrix) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying alignment engine: BLASTn-style exact-match detection (EXC) and BLASTp alignments whose raw scores feed the BSR-based classification of non-exact hits)

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Every genome-locus cell in allelic_profile_matrix.tsv contains either a valid integer allele identifier or one of the tags EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF
- Reported BSR threshold matches the expected default of 0.6 unless explicitly overridden
- Paralogous loci flagged by AlleleCall appear in RepeatedLoci.txt (tutorial: 20 loci) and are removed so the cleaned matrix has the expected reduced locus count (tutorial: 3128 → 3108 loci)
- Novel alleles identified during calling are added to the schema FASTA files (tutorial: 3,128 → 17,848 alleles)
- ASM/ALM assignments are consistent with the locus mode length and the configured size threshold (alleles smaller/larger than mode)

## Limitations

- Paralogous loci (NIPH/NIPHEM) carry allele-assignment uncertainty and should be removed from the schema (tutorial removes 20 such loci from 3128)
- Loci may be missed due to sequencing coverage and assembly problems, so core-genome thresholds below 100% presence (e.g. 95%) are recommended
- Allele-calling accuracy depends on correct sorting of highest-scoring BLASTp matches — a fixed bug in the select_highest_scores function (sorting by sequence length instead of alignment raw score) previously caused some sequences to go unclassified
- ASM/ALM size classification is sensitive to the size-threshold definition relative to the locus mode

## Evidence

- [readme] The allele call used the default BSR threshold of 0.6: [evidence span withheld — anchor status: external_doc]
- [readme] RepeatedLoci paralog list: [evidence span withheld — anchor status: external_doc]
- [discussion] Allelic profile output: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
- [discussion] ASM/ALM size threshold: "Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode)."
