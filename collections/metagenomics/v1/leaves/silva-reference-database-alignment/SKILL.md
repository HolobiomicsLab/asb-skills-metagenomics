---
name: silva-reference-database-alignment
description: Use when a set of inferred 16S rRNA sequence variants (e.g., HashSeq
  sequences.fasta or DADA2 variant sequences) must be validated against a reference
  database by aligning each variant to SILVA and computing percent identity.
when_to_use_negative:
- Input is raw fastq reads rather than already-called variant sequences — variant
  inference (e.g., inferTrueSequences with threshold=1000) must precede reference
  alignment.
- Taxonomic classification is the goal (assigning lineages) rather than validating
  that variants correspond to previously observed sequences.
- Long-read data (e.g., PacBio) where multiple-mismatch variants are common and HashSeq's
  one-mismatch assumption underlying the variant set does not hold.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0632
tools:
- name: HashSeq
  role: Produces the sequences.fasta variant set that is aligned against SILVA132;
    installed via devtools::install_github("FarnazFouladi/HashSeq")
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Provides a comparison set of variant sequences (error modeled as a function
    of quality scores per nucleotide transition) aligned to SILVA alongside HashSeq
    variants
- name: R
  role: Environment for running HashSeq and downstream percent-identity tabulation;
    BLAST scripts (codes/blast/blast.R) live in the manuscript repo
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:a969503ba9b20c6341417642b963a84820ae9bb1a19092ccf79031a7f46b9217
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
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# silva-reference-database-alignment

## Summary

Maps sequence variants called from 16S rRNA gene amplicon data (e.g., HashSeq-inferred sequences.fasta entries) against the SILVA132 reference database using BLAST, computing percent identity of each variant to its top bit-score hit to verify variants are real biological sequences rather than sequencing errors.

## When to use

Use when a set of inferred 16S rRNA sequence variants (e.g., HashSeq sequences.fasta or DADA2 variant sequences) must be validated against a reference database by aligning each variant to SILVA and computing percent identity.

## When NOT to use

- Input is raw fastq reads rather than already-called variant sequences — variant inference (e.g., inferTrueSequences with threshold=1000) must precede reference alignment.
- Taxonomic classification is the goal (assigning lineages) rather than validating that variants correspond to previously observed sequences.
- Long-read data (e.g., PacBio) where multiple-mismatch variants are common and HashSeq's one-mismatch assumption underlying the variant set does not hold.

## Inputs

- sequences.fasta of HashSeq-inferred variants (parents above the 1000-read threshold plus significant one-mismatch children)
- DADA2-inferred variant sequences (for caller comparison)
- SILVA132 reference database formatted for BLASTN

## Outputs

- silva_identity_table.csv with per-variant caller, top-hit SILVA accession, and percent identity
- Fraction of HashSeq variants with >99% identity to their top SILVA hit (Fig. 6-style summary)

## How to apply

Start from the variant sequences produced by variant calling (HashSeq's sequences.fasta after running inferTrueSequences with threshold=1000 on pre-processed PRJEB24409 mock-community fastq reads, and/or DADA2 variants). Align every variant sequence against the SILVA132 database with BLASTN and retain only the top bit-score hit per variant. Compute percent identity of each variant to its top SILVA hit as identical positions over the aligned length (equation-6 percent-identity formula). Tabulate per-variant caller, top-hit SILVA accession, and percent identity, then summarize the fraction of HashSeq variants exceeding 99% identity. Correct application is evidenced when the great majority of HashSeq variants show >99% identity to their top SILVA hit, reproducing the paper's Table 1 / Fig. 6 pattern that variants have a good match to existing SILVA entries.

## Related tools

- **HashSeq** (Produces the sequences.fasta variant set that is aligned against SILVA132; installed via devtools::install_github("FarnazFouladi/HashSeq")) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Provides a comparison set of variant sequences (error modeled as a function of quality scores per nucleotide transition) aligned to SILVA alongside HashSeq variants)
- **R** (Environment for running HashSeq and downstream percent-identity tabulation; BLAST scripts (codes/blast/blast.R) live in the manuscript repo) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
devtools::install_github("FarnazFouladi/HashSeq"); inferTrueSequences(inputDir, outputDir, threshold=1000)  # then BLASTN sequences.fasta against SILVA132 and compute percent identity
```

## Evaluation signals

- The great majority of HashSeq variants show >99% percent identity to their top bit-score SILVA132 hit, matching the paper's Table 1 / Fig. 6 pattern.
- silva_identity_table.csv has exactly one top-hit SILVA accession per variant (top bit-score hit retained, not all hits).
- Percent identities are computed over the aligned length (identical positions / aligned positions), consistent with the equation-6 formula.
- Both HashSeq and DADA2 variant sets are represented in the table so callers can be compared on identity to SILVA.
- Variant input sequences trace back to HashSeq outputs above the 1000-read abundance threshold (sequences.fasta / SvTable.txt).

## Limitations

- High SILVA identity supports that variants were previously observed but is an indirect validation; HashSeq is conservative and not sensitive for true low-abundance sequences, so such variants may never reach the alignment stage.
- HashSeq considers only one-mismatch variants and does not model indels (indel events are treated as separate parent sequences), which shapes which variants exist to be aligned.
- The one-mismatch assumption may not hold for long-read technologies such as PacBio, limiting generalization of the variant set being validated.

## Evidence

- [readme] Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases."
- [discussion] SILVA match finding: "This conservative approach detected sequence variants that had a good match to existing variants in the SILVA database"
- [readme] HashSeq variant output: "sequences.fasta: This file includes all parents above the threshold and the significant one-mismatch children from step 3."
- [methods] Mock community dataset: "three mock communities of 16S rRNA gene sequences that are publicly available at NCBI under project# PRJEB24409"
- [intro] DADA2 error model comparison: "DADA2 models error rate as a function of quality scores for each possible nucleotide transition"
