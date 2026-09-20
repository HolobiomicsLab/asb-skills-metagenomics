---
name: markov-order-model-selection
description: Use when running CAFE background-adjusted measures (CVtree, D2star, D2shepp)
  on genomes or NGS shotgun reads and the appropriate Markov order is unclear to the
  user.
when_to_use_negative:
- Using the JS (Jensen-Shannon) distance in CAFE — BIC-based automatic order inference
  is explicitly 'not suitable for JS'
- The user already knows the appropriate Markov order (e.g. order 0 for the gut metagenome
  d2S analysis) — fixing -M directly is faster than the time-consuming BIC search
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: CAFE
  role: Runs the BIC-based Markov order inference via the -M -1 option when calculating
    background-adjusted measures (D2star, D2shepp, CVtree)
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Accelerates k-mer counting (invoked through CAFE's -J option with the jellyfish
    executable path, e.g. jellyfish-2.2.4/bin/./jellyfish) while orders are being
    evaluated
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:17527b9977a66e4f80de4e69b0dc4d53033096a34311216cc381d284bdd730f9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# markov-order-model-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic inference of the Markov model order required by CAFE's background-adjusted alignment-free measures (CVtree, D2star/D2*, D2shepp/d2S) by setting the -M parameter to -1, which selects the order minimizing the Bayesian Information Criterion (BIC) from the sequence data itself.

## When to use

Use when running CAFE background-adjusted measures (CVtree, D2star, D2shepp) on genomes or NGS shotgun reads and the appropriate Markov order is unclear to the user.

## When NOT to use

- Using the JS (Jensen-Shannon) distance in CAFE — BIC-based automatic order inference is explicitly 'not suitable for JS'
- The user already knows the appropriate Markov order (e.g. order 0 for the gut metagenome d2S analysis) — fixing -M directly is faster than the time-consuming BIC search

## Inputs

- FASTA sequence files ('.fasta', '.fa', '.fna') of genomes or NGS shotgun reads
- K-mer length K (background-adjusted order must be a non-negative integer less than K, or -1)

## Outputs

- Inferred optimal Markov order per sequence (selected by minimizing BIC)
- Pairwise dissimilarity matrix computed at the inferred order (plain or PHYLIP format, e.g. result.D2shepp.phylip)
- markov_orders.csv with dataset and inferred_order columns

## How to apply

Load FASTA inputs (extensions '.fasta', '.fa' or '.fna') into CAFE and set the Markov order parameter to -1 (the default), which triggers automatic inference of the optimal order via BIC; the order candidate is kept below the k-mer length K. Optionally accelerate k-mer counting with JELLYFISH via the -J option (e.g. jellyfish-2.2.4 binary path). CAFE evaluates the BIC for each candidate Markov order and selects the order minimizing the criterion, then computes the background-adjusted k-mer counts for the chosen order. CAFE also accepts a single fixed order or a comma-separated list of per-sequence orders, so -M -1 should only be used when the order is genuinely unknown; note BIC-based inference is 'not suitable for JS'. Record the inferred orders per dataset and validate against known values (e.g. 12 for primate genomes, 1 for E. coli/Shigella, 0 for gut metagenome reads in the paper's datasets).

## Related tools

- **CAFE** (Runs the BIC-based Markov order inference via the -M -1 option when calculating background-adjusted measures (D2star, D2shepp, CVtree)) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Accelerates k-mer counting (invoked through CAFE's -J option with the jellyfish executable path, e.g. jellyfish-2.2.4/bin/./jellyfish) while orders are being evaluated)

## Examples

```
./cafe -M -1 -D D2star -K 10 -I speciesA.fa,speciesB.fa -J jellyfish-2.2.4/bin/./jellyfish -T phylip -O output_path
```

## Evaluation signals

- The inferred_order for primate genomes equals 12, for E. coli/Shigella genomes equals 1, and for gut metagenome reads equals 0 (direct field checks in markov_orders.csv)
- Every inferred order is a non-negative integer strictly less than the k-mer length K
- The BIC evaluated across candidate orders is minimized by the selected order (candidate search bounded below K)
- Downstream dissimilarity matrices (e.g. result.D2shepp.phylip in PHYLIP format) are produced successfully with the inferred order substituted for -1

## Limitations

- BIC-based order inference is time-consuming compared with fixing the order directly
- Automatic inference is not suitable for the JS (Jensen-Shannon) measure
- The Windows default executable cafe_win.exe may not work as expected due to User Account Control permission settings; rename cafe_win_2.exe / cafe_win_3.exe if errors occur

## Evidence

- [other] run CAFE with the Markov order option set to -1, which triggers automatic inference of the suitable order via the Bayesian Information Criterion (BIC), keeping the order candidate below the k-mer length K: [evidence span withheld — anchor status: not_found]
