---
name: bit-score-best-hit-selection
description: Use when you have a set of inferred amplicon sequence variants (e.g.,
  HashSeq sequences.fasta) and need to confirm they match previously observed references
  by BLASTing against SILVA132 and keeping the top bit-score hit per variant.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been variant-called (pre-process and
  run inferTrueSequences first)
- Input is already a taxonomic assignment or feature table rather than variant sequences
  needing reference validation
- Variants contain insertions/deletions you care about — HashSeq does not explicitly
  model indels, so percent identity over a single aligned variant may not capture
  indel variation; the paper recommends Deblur or alignment-based methods in that
  case
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0091
tools:
- name: HashSeq
  role: Produces the inferred variant sequences.fasta that are BLASTed against SILVA132
    to assess reference support
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Comparison caller whose variants are also BLASTed against SILVA132 to contrast
    identity profiles
- name: R
  role: Runs codes/blast/blast.R to perform BLASTN searches, pick top bit-score hits,
    and tabulate percent identities
  repo: https://github.com/FarnazFouladi/HashSeq_Manuscript
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:50316611a77b2a0520635e03fda34b2f3b71e2ce319113cc8562ce14e752eb16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bit-score-best-hit-selection

## Summary

Selects, for each HashSeq- or DADA2-inferred 16S rRNA sequence variant, the single best SILVA132 BLASTN hit by top bit score and computes percent identity (identical positions over aligned length) to validate that inferred variants are real biological sequences rather than sequencing errors.

## When to use

Use when you have a set of inferred amplicon sequence variants (e.g., HashSeq sequences.fasta) and need to confirm they match previously observed references by BLASTing against SILVA132 and keeping the top bit-score hit per variant.

## When NOT to use

- Input is raw FASTQ reads that have not yet been variant-called (pre-process and run inferTrueSequences first)
- Input is already a taxonomic assignment or feature table rather than variant sequences needing reference validation
- Variants contain insertions/deletions you care about — HashSeq does not explicitly model indels, so percent identity over a single aligned variant may not capture indel variation; the paper recommends Deblur or alignment-based methods in that case

## Inputs

- sequences.fasta of HashSeq-inferred true sequence variants (parents above threshold plus significant one-mismatch children)
- DADA2-inferred variant sequences for the same PRJEB24409 mock-community data
- SILVA132 reference database formatted for BLASTN

## Outputs

- silva_identity_table.csv listing per-variant caller, top-hit SILVA accession, and percent identity
- Fig. 6-style summary of the fraction of HashSeq variants with >99% identity to their top SILVA hit

## How to apply

Align every inferred variant sequence against the SILVA132 reference database with BLASTN (e.g., via the codes/blast/blast.R script from the HashSeq_Manuscript repository) and retain only the top bit-score hit for each variant. For each retained hit, compute percent identity using the equation-6 formula: identical positions divided by the aligned length. Tabulate per-variant caller (HashSeq vs DADA2), top-hit SILVA accession, and percent identity into silva_identity_table.csv, then summarize the fraction of HashSeq variants exceeding 99% identity. The validation criterion from the paper is that the great majority of HashSeq variants show >99% identity to their top SILVA hit, indicating the conservative caller produces variants that have a good match to existing variants in SILVA.

## Related tools

- **HashSeq** (Produces the inferred variant sequences.fasta that are BLASTed against SILVA132 to assess reference support) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Comparison caller whose variants are also BLASTed against SILVA132 to contrast identity profiles)
- **R** (Runs codes/blast/blast.R to perform BLASTN searches, pick top bit-score hits, and tabulate percent identities) — https://github.com/FarnazFouladi/HashSeq_Manuscript

## Examples

```
Rscript codes/blast/blast.R
```

## Evaluation signals

- Exactly one top bit-score SILVA132 hit is retained per variant (no ties or multiple hits per variant in silva_identity_table.csv)
- Percent identity is computed as identical positions over the aligned length (equation-6 formula) for every variant
- The great majority of HashSeq variants show >99% identity to their top SILVA hit, reproducing the Table 1 / Fig. 6 pattern
- Each row records the caller (HashSeq or DADA2), top-hit accession, and percent identity with no missing accessions
- Output silva_identity_table.csv and the Fig. 6-style summary are both produced

## Limitations

- Identity to SILVA is support for, not proof of, biological realism — the conservative HashSeq approach filters low-abundance variants and is not sensitive for detecting true low-abundance sequences
- HashSeq considers only one-mismatch variants and does not model indels, so alignment-based percent identity may not capture insertion/deletion variation
- The assumption of rarity of multiple-mismatch sequences may not hold for long-read technologies such as PacBio

## Evidence

- [discussion] This conservative approach detected sequence variants that had a good match to existing variants in the SILVA database: "This conservative approach detected sequence variants that had a good match to existing variants in the SILVA database"
- [readme] Run the blast analysis script: [evidence span withheld — anchor status: not_found]
- [intro] HashSeq infers true variants relative to background error: "Our approach provides a conservative set of variant calls that is well supported by a reference database"
- [intro] DADA2 error model description: "DADA2 models error rate as a function of quality scores for each possible nucleotide transition"
