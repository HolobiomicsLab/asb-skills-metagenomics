---
name: phred-like-quality-score-computation
description: Use when you have per-alignment contig-to-reference-replicon identities
  (contigs above the --minLength threshold) from an assembly benchmark like LMAS and
  need a per-contig, Phred-like quality score comparable across assemblers.
when_to_use_negative:
- Contigs shorter than the --minLength threshold (1000 bp) — they are excluded from
  the alignment input and therefore from Pls computation
- Assemblies with no reference replicons available — the score requires contig-to-reference
  alignment identity and cannot be computed from contig features alone (unlike QUAST-style
  reference-free metrics)
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Automated Nextflow workflow that generates contig-to-reference alignments
    and computes the per-contig Pls (Phred-like) score as part of its assembly evaluation
    stage
  repo: https://github.com/B-UMMI/LMAS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:d6142e7c31dcce654fdbb074198508d9eb07f86c70ac528aa93708f4fe260a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# phred-like-quality-score-computation

## Summary

Computes the LMAS Pls (Phred-like) assembly quality score per contig by converting contig-to-reference identity into a Phred-style value, following Equation 1 with E = 1 - Identity as the error function and a cap of 60 when E = 0.

## When to use

Use when you have per-alignment contig-to-reference-replicon identities (contigs above the --minLength threshold) from an assembly benchmark like LMAS and need a per-contig, Phred-like quality score comparable across assemblers.

## When NOT to use

- Contigs shorter than the --minLength threshold (1000 bp) — they are excluded from the alignment input and therefore from Pls computation
- Assemblies with no reference replicons available — the score requires contig-to-reference alignment identity and cannot be computed from contig features alone (unlike QUAST-style reference-free metrics)

## Inputs

- Contig-to-reference alignment results per assembler (per-alignment identity of contigs over the minimum contig size threshold of 1000 bp)
- Reference replicon sequences from the defined mock community (e.g. ZymoBIOMICS, BMock12, Gut-Mix-RR, Gut-Mix-HiLo)
- Assembler identity tag and minimum contig length (--minLength) parameter value

## Outputs

- pls_scores.csv with one Phred-like Pls value per aligned contig, tagged with assembler and reference replicon

## How to apply

Load the contig-to-reference alignment results for each assembler, keeping only contigs over the minimum contig size threshold (1000 bp, set via the --minLength parameter) that map to a reference replicon. For each aligned contig, compute the error rate E = 1 - Identity, where Identity is the fraction of the alignment matching the reference. Convert this to a Phred-like score, PLS = -10 * log10(E), applying the special-case cap of 60 when E = 0. Write the per-contig PLS values to pls_scores.csv, tagged with the assembler and reference replicon, so scores can be compared across assemblers in the LMAS interactive report.

## Related tools

- **LMAS** (Automated Nextflow workflow that generates contig-to-reference alignments and computes the per-contig Pls (Phred-like) score as part of its assembly evaluation stage) — https://github.com/B-UMMI/LMAS

## Examples

```
pls = 60 if identity == 1.0 else -10 * math.log10(1 - identity)  # E = 1 - Identity, capped at 60 when E = 0
```

## Evaluation signals

- pls_scores.csv exists and contains a Pls value for every aligned contig above the 1000 bp --minLength threshold
- No score exceeds the cap of 60 (the special case for E = 0)
- Spot-check: a contig with alignment identity 0.99 must yield Pls = -10 * log10(0.01) = 20
- Every entry in pls_scores.csv is tagged with both an assembler name and a reference replicon

## Limitations

- Score is bounded at 60, so perfect alignments (E = 0) cannot be distinguished from each other by higher values
- Only contigs mapping to known reference replicons and passing the minimum contig size filter receive a score; unresolved or non-mapping contigs are not scored
- Meaningful strain resolution at the single-nucleotide polymorphism level was not achieved by any assembler tested, so Pls should not be read as a strain-level accuracy guarantee

## Evidence

- [methods] The Pls is a Phred-like score computed per contig using as the error function the identity of each aligned contig to the reference replicon: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [discussion] LMAS integrates assembly and evaluation in one pipeline, ensuring comparable Pls computation across assemblers: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [intro] Pls scores are surfaced in the reference replicon-specific section of the interactive HTML report: "The results of LMAS are presented in an interactive HTML report where selected global and reference replicon-specific performance metrics can be explored."
- [methods] LMAS source repository used for the Pls implementation: "LMAS is open source and available at https://github.com/cimendes/LMAS"
