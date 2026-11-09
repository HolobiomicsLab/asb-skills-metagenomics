---
name: markov-background-model-estimation
description: Use when computing background-adjusted alignment-free distances (CVtree,
  D2star, D2shepp) in CAFE and the sequence's Markov order is unknown — the default
  '-1' triggers automatic BIC-based inference.
when_to_use_negative:
- Running conventional k-mer count measures (Ma, Eu, Ch, etc.) or presence/absence
  measures (Jaccard, Dice, etc.) — these do not require a Markov background model
- Using -M -1 automatic inference with the JS (Jensen-Shannon) measure — the README
  states the '-1' BIC inference is 'not suitable for JS'
- When a reliable Markov order is already known for all sequences and BIC estimation
  would add unnecessary runtime
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: CAFE
  role: Standalone executable (./cafe) that applies the Markov order via the -M option
    to compute expected background k-mer counts for D2star, D2shepp, and CVtree
  repo: https://github.com/younglululu/CAFE
- name: jellyfish
  role: Accelerates the k-mer counting step (-J <jfexe_path>) on which the Markov-order-corrected
    counts are based
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:dd05ced79d5fdb8632e8ea7bdadb188b6dd9ba534d4f36f3e97460bad1a9fe6d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/younglululu/CAFE
  repo_url: https://github.com/younglululu/CAFE
  tools:
  - CAFE
  - JELLYFISH
  - jellyfish
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# markov-background-model-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate the appropriate Markov order (background sequence model) used by CAFE's background-adjusted k-mer dissimilarity measures (D2star, D2shepp, CVtree), either per-sequence via the Bayesian Information Criterion (BIC) or with user-specified order values.

## When to use

Use when computing background-adjusted alignment-free distances (CVtree, D2star, D2shepp) in CAFE and the sequence's Markov order is unknown — the default '-1' triggers automatic BIC-based inference.

## When NOT to use

- Running conventional k-mer count measures (Ma, Eu, Ch, etc.) or presence/absence measures (Jaccard, Dice, etc.) — these do not require a Markov background model
- Using -M -1 automatic inference with the JS (Jensen-Shannon) measure — the README states the '-1' BIC inference is 'not suitable for JS'
- When a reliable Markov order is already known for all sequences and BIC estimation would add unnecessary runtime

## Inputs

- FASTA sequence files (.fasta/.fa/.fna) — assembled genomes or NGS shotgun reads
- Markov order specification for -M: single integer, comma-separated per-sequence list, or -1 for BIC inference
- k-mer length (-K) constraining the allowable Markov order (order must be < K)

## Outputs

- Background-adjusted (Markov-corrected) expected k-mer counts feeding D2star, D2shepp, and CVtree dissimilarity computations
- Pairwise dissimilarity matrices (e.g. PHYLIP format via -T phylip) incorporating the inferred/specified Markov order

## How to apply

Set the Markov order with the CAFE -M option when running measures that require expected background k-mer counts (D2star, D2shepp, CVtree). Pass a single non-negative integer so all sequences share the same order, a comma-separated list matching the number of input FASTA files for per-sequence orders, or '-1' (the default) to infer each sequence's optimal order automatically via the Bayesian Information Criterion. The order must be a non-negative integer strictly less than the k-mer length (-K). Note that '-1' automatic inference is explicitly not suitable for JS (Jensen-Shannon divergence). Rationale: the original d2*/d2S implementations are slow because of expected k-mer count computation, so CAFE accelerates this background adjustment while keeping the Markov model formulation.

## Related tools

- **CAFE** (Standalone executable (./cafe) that applies the Markov order via the -M option to compute expected background k-mer counts for D2star, D2shepp, and CVtree) — https://github.com/younglululu/CAFE
- **jellyfish** (Accelerates the k-mer counting step (-J <jfexe_path>) on which the Markov-order-corrected counts are based)

## Examples

```
./cafe -M -1 -O result.phylip -T phylip -I speciesA.fa,speciesB.fa -J jellyfish-2.2.4/bin/./jellyfish -K 10 -D D2star,CVtree
```

## Evaluation signals

- Confirm the -M value is either -1, a non-negative integer, or a comma-separated list whose length matches the number of input FASTA files
- Verify the specified/inferred Markov order is strictly less than the k-mer length given by -K
- For BIC inference (-1), check the CAFE console/running-information panel reports the per-sequence inferred orders during execution
- Re-run with a fixed known order (e.g. -M 0 as in the README examples) and confirm the resulting PHYLIP matrix reproduces the same distances
- Confirm background-adjusted measures (D2star, D2shepp, CVtree) computed with the inferred order outperform non-adjusted measures in downstream correlation with evolutionary distances

## Limitations

- Automatic Markov order inference via BIC ('-1') is time-consuming ('the simple yet time-consuming way') and is not suitable for JS
- The Markov order must be less than the k-mer length; a comma-separated order list must exactly match the number of input FASTA files
- Expected k-mer count computation for background-adjusted measures was the bottleneck in original d2*/d2S implementations, motivating CAFE's accelerated version

## Evidence
