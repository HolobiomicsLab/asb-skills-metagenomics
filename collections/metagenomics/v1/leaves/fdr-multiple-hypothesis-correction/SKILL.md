---
name: fdr-multiple-hypothesis-correction
description: Use when interpreting HashSeq's per-variant inference-test P values (e.g.
  deciding which one-mismatch children are true variants) and you need FDR-adjusted
  significance across many clusters tested simultaneously.
when_to_use_negative:
- The task requires detecting true low-abundance sequence variants — the paper states
  HashSeq is not sensitive for these and recommends more sensitive algorithms such
  as DADA2.
- The variant calls must capture indels or two-or-more-mismatch differences relative
  to a parent — HashSeq considers only one-mismatch variants and treats indels as
  separate parent sequences.
- Data come from long-read technologies (e.g. PacBio) where the assumption of rarity
  of multiple-mismatch sequences may not hold.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: HashSeq
  role: Generates per-variant inference-test P values from one-mismatch clusters whose
    background error is modeled with a normal distribution and LOESS regression; corrected
    P values feed its variant-calling decisions
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Implementation environment for HashSeq and for applying Benjamini-Hochberg
    FDR correction to the inference-test P values
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:96143dbde87c70a39ee267ac0d3241c4cde26136fe3b27b20fe9b217cc0df698
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fdr-multiple-hypothesis-correction

## Summary

Apply Benjamini-Hochberg false discovery rate (FDR) correction to the inference-test P values produced by HashSeq so that one-mismatch children flagged as significant are controlled for multiple testing across the large number of sequence-variant clusters.

## When to use

Use when interpreting HashSeq's per-variant inference-test P values (e.g. deciding which one-mismatch children are true variants) and you need FDR-adjusted significance across many clusters tested simultaneously.

## When NOT to use

- The task requires detecting true low-abundance sequence variants — the paper states HashSeq is not sensitive for these and recommends more sensitive algorithms such as DADA2.
- The variant calls must capture indels or two-or-more-mismatch differences relative to a parent — HashSeq considers only one-mismatch variants and treats indels as separate parent sequences.
- Data come from long-read technologies (e.g. PacBio) where the assumption of rarity of multiple-mismatch sequences may not hold.

## Inputs

- Raw P values from HashSeq's inference test for one-mismatch children (childrenProperties.txt / InferenceTestSignificantChildren.txt context)
- Count table (SvTable.txt) produced by inferTrueSequences with abundance threshold (e.g. 1,000 reads)
- FDR threshold choice (e.g. 5% / fdr=0.05)

## Outputs

- FDR-adjusted P values for each candidate sequence variant
- Set of significant one-mismatch children retained in InferenceTestSignificantChildren.txt, sequences.fasta, and SvTable.txt

## How to apply

HashSeq evaluates each one-mismatch child against a normal/LOESS-modeled background error rate as a function of parent sequencing depth, yielding a raw P value per variant. Because thousands of clusters are tested at once, raw P values must be adjusted with Benjamini-Hochberg FDR correction before declaring variants true; in the mock-community (zymo) example this is applied downstream of inferTrueSequences(inputDir, outputDir, threshold=1000) at fdr=0.05. After correction, only significant one-mismatch children are retained in InferenceTestSignificantChildren.txt, sequences.fasta, and SvTable.txt alongside their parent sequences. The paper recommends feeding the corrected P values into downstream statistical models rather than interpreting uncorrected P values.

## Related tools

- **HashSeq** (Generates per-variant inference-test P values from one-mismatch clusters whose background error is modeled with a normal distribution and LOESS regression; corrected P values feed its variant-calling decisions) — https://github.com/FarnazFouladi/HashSeq
- **R** (Implementation environment for HashSeq and for applying Benjamini-Hochberg FDR correction to the inference-test P values) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000, fdr=0.05)
```

## Evaluation signals

- At 5% FDR, the set of significant one-mismatch children on the zymo mock community should be small (README example: 26 of 230 clusters with children), consistent with a conservative caller.
- Parent sequences retained after FDR filtering should include the eight bacterial taxa present in the Zymo mock community, confirming major taxa are recovered.
- Variant counts at 5% FDR should be lower than (more conservative than) DADA2's default calls on the same trimmed 250-base mock reads.
- Corrected P values should be used (never raw P values) as inputs to downstream statistical models.
- makeLoessPlot on childrenProperties.txt should show the LOESS regression is a good fit above the abundance threshold before trusting error-model P values.

## Limitations

- The inference test considers only one-mismatch variants; two or more mismatches are not evaluated.
- HashSeq is not sensitive for detecting true low-abundance sequences; more sensitive algorithms such as DADA2 are recommended for low-abundance variants.
- Indels are not explicitly modeled and indel events are treated as separate parent sequences.
- The Java virtual machine default heap (512MB) may be insufficient for large data sets; increase via options(java.parameters) before attaching libraries.

## Evidence

- [discussion] Adjusting P values with Benjamini-Hochberg FDR multiple-hypothesis correction in downstream analyses: "which can be fed into statistical models to produce FDR-corrected P values"
- [abstract] Normal/LOESS error model underlying the inference test: "use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth"
- [readme] FDR=0.05 variant calling on the mock community: "InferenceTestSignificantChildren.txt: This file includes one-mismatch children that were significant in an inference test."
- [readme] Abundance filtering threshold: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [discussion] Conservativeness relative to DADA2: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true."
