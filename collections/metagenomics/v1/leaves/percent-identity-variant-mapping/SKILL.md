---
name: percent-identity-variant-mapping
description: Use when inferred sequence variants (FASTA from a variant caller) need
  validation against a reference database such as SILVA132 to confirm they match previously
  observed sequences with high identity (>99%).
when_to_use_negative:
- Input is a raw fastq file that has not yet been processed by a variant caller (primers
  removed, reads paired, trimmed to 250 bases) — this skill validates inferred variants,
  not raw reads.
- The goal is detecting novel or low-abundance variants; HashSeq is conservative and
  not sensitive for true low-abundance sequences, so low-identity or novel-variant
  discovery is out of scope.
- Sequences with two or more mismatches to a parent or indel variation are not modeled
  by HashSeq, so identity mapping cannot support claims about indel-containing variants.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: HashSeq
  role: Produces the inferred variant sequences (sequences.fasta, SvTable.txt) that
    are mapped to SILVA132
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Comparison caller whose variants are aligned against SILVA132 in parallel
    to assess caller agreement
- name: R
  role: Environment for running HashSeq and downstream tabulation/summarization (silva_identity_table.csv,
    Fig. 6-style summary)
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:90982e2b1185cde82b2c29982ca32cc733ca166a4cc169c2188d70c564ea4195
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# percent-identity-variant-mapping

## Summary

Validates de novo inferred 16S rRNA sequence variants (from HashSeq or DADA2) by aligning each variant against the SILVA132 reference database (e.g., BLASTN), keeping the top bit-score hit, and computing percent identity (identical positions over aligned length) to judge whether variants are real biological sequences rather than sequencing errors.

## When to use

Use when inferred sequence variants (FASTA from a variant caller) need validation against a reference database such as SILVA132 to confirm they match previously observed sequences with high identity (>99%).

## When NOT to use

- Input is a raw fastq file that has not yet been processed by a variant caller (primers removed, reads paired, trimmed to 250 bases) — this skill validates inferred variants, not raw reads.
- The goal is detecting novel or low-abundance variants; HashSeq is conservative and not sensitive for true low-abundance sequences, so low-identity or novel-variant discovery is out of scope.
- Sequences with two or more mismatches to a parent or indel variation are not modeled by HashSeq, so identity mapping cannot support claims about indel-containing variants.

## Inputs

- sequences.fasta of HashSeq-inferred variants (parents above the abundance threshold plus significant one-mismatch children)
- DADA2 variant sequences for the same input data
- SILVA132 reference database for BLASTN alignment
- Pre-processed PRJEB24409 mock-community fastq files (primers removed, forward/reverse reads paired, trimmed to 250 bases)

## Outputs

- silva_identity_table.csv with per-variant caller, top-hit SILVA accession, and percent identity
- Fig. 6-style summary of the fraction of variants with >99% identity to their top SILVA hit

## How to apply

After variant calling (e.g., HashSeq inferTrueSequences with threshold=1000 producing sequences.fasta, or DADA2 on the same pre-processed PRJEB24409 mock-community fastq data), align every variant sequence against SILVA132 using BLASTN and retain the top bit-score hit per variant. Compute percent identity for each variant to its top hit using the equation-6 formula: identical positions over the aligned length. Tabulate per-variant caller, top-hit SILVA accession, and percent identity (silva_identity_table.csv), then summarize the fraction of HashSeq variants exceeding 99% identity. The rationale is that HashSeq's conservative, normal-distribution/LOESS-based error model should call variants with a good match to existing SILVA variants.

## Related tools

- **HashSeq** (Produces the inferred variant sequences (sequences.fasta, SvTable.txt) that are mapped to SILVA132) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Comparison caller whose variants are aligned against SILVA132 in parallel to assess caller agreement)
- **R** (Environment for running HashSeq and downstream tabulation/summarization (silva_identity_table.csv, Fig. 6-style summary)) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
inferTrueSequences(inputDir, outputDir, threshold=1000)  # then BLASTN outputDir/sequences.fasta against SILVA132 and tabulate top-hit percent identity
```

## Evaluation signals

- The great majority of HashSeq variants show >99% identity to their top SILVA132 hit, matching the paper's Table 1 / Fig. 6 pattern.
- Each variant has exactly one retained top bit-score hit accession in silva_identity_table.csv.
- Percent identity values are computed as identical positions over the aligned length (equation-6 formula).
- HashSeq and DADA2 variants from the same pre-processed input are both represented in the table, allowing caller-level comparison.
- Variant sequences in the table correspond to entries in sequences.fasta and SvTable.txt from inferTrueSequences.

## Limitations

- A good SILVA match shows variants were previously observed, but does not prove correct taxonomy or quantify novel-diversity sensitivity.
- HashSeq does not model indels or two-or-more-mismatch variants, so identity assessment is limited to substitution-only one-mismatch relationships.
- HashSeq is conservative and not sensitive for true low-abundance sequences; low-abundance variants may be absent from the identity table entirely.

## Evidence

- [discussion] The conservative approach detected sequence variants that had a good match to existing variants in the SILVA database: "This conservative approach detected sequence variants that had a good match to existing variants in the SILVA database"
- [methods] Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases"
- [readme] threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [readme] sequences.fasta includes all parents above the threshold and the significant one-mismatch children: "sequences.fasta: This file includes all parents above the threshold and the significant one-mismatch children from step 3."
- [discussion] HashSeq is not sensitive to detect true low-abundance sequences; recommend DADA2 for low-abundance variants: "our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2"
