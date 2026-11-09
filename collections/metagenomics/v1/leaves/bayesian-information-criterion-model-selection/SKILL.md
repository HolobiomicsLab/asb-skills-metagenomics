---
name: bayesian-information-criterion-model-selection
description: Use when running CAFE's background-adjusted k-mer measures (CVtree, D2star,
  D2shepp) on FASTA sequences and the appropriate Markov order encoding the sequence
  model is unclear to the user.
when_to_use_negative:
- When using the JS (Jensen-Shannon divergence) measure, for which the '-1' automatic
  order inference is explicitly not suitable
- When the Markov order is already known — pass a non-negative integer less than K
  (e.g. -M 0) instead of paying the time cost of BIC inference
- When the analysis is alignment-based (e.g. BLAST against RefSeq) rather than an
  alignment-free k-mer comparison requiring a background sequence model
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CAFE
  role: Standalone executable that implements the -M/-1 BIC-based automatic Markov
    order inference for its background-adjusted k-mer distance measures
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Optional accelerator for the k-mer counting step, invoked via CAFE's -J option
    (e.g. jellyfish-2.2.4 binary path)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:a867f4560f13c7dba6a47b3943d747e06543bf49c4f7e2fd216151a53831e398
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bayesian-information-criterion-model-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic inference of the optimal Markov model order for background-adjusted alignment-free measures (CVTree, D2star/D2shepp) by setting the CAFE Markov order option to '-1', which evaluates candidate orders (each below the k-mer length K) with the Bayesian Information Criterion (BIC).

## When to use

Use when running CAFE's background-adjusted k-mer measures (CVtree, D2star, D2shepp) on FASTA sequences and the appropriate Markov order encoding the sequence model is unclear to the user.

## When NOT to use

- When using the JS (Jensen-Shannon divergence) measure, for which the '-1' automatic order inference is explicitly not suitable
- When the Markov order is already known — pass a non-negative integer less than K (e.g. -M 0) instead of paying the time cost of BIC inference
- When the analysis is alignment-based (e.g. BLAST against RefSeq) rather than an alignment-free k-mer comparison requiring a background sequence model

## Inputs

- FASTA sequence files ('.fasta', '.fa', '.fna'), either long genomic sequences or unassembled NGS shotgun reads
- K-mer length K (-K option) for the chosen distance measure
- Distance measure selection (e.g. CVtree, D2star, D2shepp) via the -D option

## Outputs

- Automatically inferred optimal Markov order for each input sequence
- Pairwise dissimilarity matrix (e.g. result.Ma.phylip or -O output path) computed with the inferred order
- Task-specific record: markov_orders.csv with columns dataset and inferred_order

## How to apply

Load the input FASTA files ('.fasta', '.fa' or '.fna') with the CAFE executable and invoke it with the -M (Markov Order) option set to -1, which triggers automatic inference of the suitable order via the BIC; CAFE counts k-mers (optionally accelerated with the -J option pointing to the jellyfish 2.2.4 executable) and evaluates the BIC for each candidate Markov order, keeping candidates below the k-mer length K, selecting the order minimizing the criterion. The inferred order is then applied for computing the background adjusted k-mer counts of the selected distance measure. Note that BIC-based automatic inference is documented as the simple yet time-consuming option, so an explicit non-negative order less than K should be used when the order is already known.

## Related tools

- **CAFE** (Standalone executable that implements the -M/-1 BIC-based automatic Markov order inference for its background-adjusted k-mer distance measures) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Optional accelerator for the k-mer counting step, invoked via CAFE's -J option (e.g. jellyfish-2.2.4 binary path))

## Examples

```
./cafe -M -1 -O output_path -T plain -I speciesA.fa,speciesB.fa -J jellyfish-2.2.4/bin/./jellyfish -K 10 -D D2star
```

## Evaluation signals

- The inferred_order recorded per dataset matches the reported validation values: primate genomes = 12, E. coli/Shigella genomes = 1, gut metagenome reads = 0
- markov_orders.csv exists with exactly the columns 'dataset' and 'inferred_order', one row per input dataset
- Every inferred order is a non-negative integer strictly less than the k-mer length K supplied via -K
- Pairwise distance output (phylip format) is produced for all input sequences using the inferred order, and the run completes without order-related errors in the CAFE console

## Limitations

- Automatic BIC-based order inference is time-consuming compared to specifying the order directly
- The '-1' option is not suitable for the JS (Jensen-Shannon divergence) measure
- The Markov order option is relevant only to D2star, D2shepp and JS (and CVTree-style background-adjusted measures), not the presence/absence k-mer measures
- No license information or software version number is stated for CAFE in the available documentation

## Evidence
