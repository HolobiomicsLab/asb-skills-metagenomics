---
name: size-accuracy-tradeoff-visualization
description: 'Decision support for choosing a counting Bloom filter size: when per-read
  abundance outputs (fimpera-wrapped and raw cBF) plus ground-truth abundances are
  available and the goal is to quantify how many bits each method needs to reach a
  target overestimation score.'
when_to_use_negative:
- Abundances were indexed with the log2 grouping strategy (abundanceToIdentifier surjective
  functions), since the score must be computed with original non-log abundance storage
- 'Errors other than overestimations matter: the score deliberately counts only squared
  overestimation differences, ignoring underestimation errors'
- The goal is k-mer-level or presence/absence evaluation rather than per-read average
  abundance comparison against ground truth
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: fimpera
  role: Produces the fimpera-wrapped per-read abundance queries against counting Bloom
    filters of varying sizes that are scored and plotted
  repo: https://github.com/lrobidou/fimpera
- name: Python
  role: Computes the sum-of-squared-error overestimation scores, tabulates size-vs-score
    pairs, and renders fig5_sse_tradeoff.png
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:1a5ad6acc7af4dc84078e2f8bdbbb65cebf06c2dbe3a56006769a1d60243ce50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - Python
  - fimpera
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# size-accuracy-tradeoff-visualization

## Summary

Reproduce the Fig. 5 space-vs-accuracy trade-off for fimpera-wrapped versus raw counting Bloom filter (cBF) queries by computing a sum-of-squared-error overestimation score from per-read average abundances across filter sizes and plotting score against filter size in bits.

## When to use

Decision support for choosing a counting Bloom filter size: when per-read abundance outputs (fimpera-wrapped and raw cBF) plus ground-truth abundances are available and the goal is to quantify how many bits each method needs to reach a target overestimation score.

## When NOT to use

- Abundances were indexed with the log2 grouping strategy (abundanceToIdentifier surjective functions), since the score must be computed with original non-log abundance storage
- Errors other than overestimations matter: the score deliberately counts only squared overestimation differences, ignoring underestimation errors
- The goal is k-mer-level or presence/absence evaluation rather than per-read average abundance comparison against ground truth

## Inputs

- Per-read average abundance outputs from fimpera-wrapped queries (across multiple filter sizes)
- Per-read average abundance outputs from raw counting Bloom filter (cBF) queries (across multiple filter sizes)
- Original ground-truth abundances (non-log storage)
- Filter size settings in bits for each method

## Outputs

- Table of overestimation score vs filter size (bits) for fimpera and cBF
- fig5_sse_tradeoff.png: size-vs-score trade-off plot with reference operating points annotated

## How to apply

Load per-read average abundance outputs from fimpera-wrapped queries, raw cBF queries, and the original ground-truth abundances, keeping abundances in original (non-log) storage. For each filter size setting, accumulate squared differences between reported and original abundances, counting only overestimation errors, to obtain the sum-of-squared-error overestimation score. Tabulate score alongside filter size in bits for both methods, then plot size-vs-score curves with fimpera and cBF as separate series and annotate the reference operating points: score 200000 at ~3.10x10^9 bits (fimpera) versus ~330.9x10^9 bits (raw cBF). Validation requires fimpera to reach the same overestimation score roughly two orders of magnitude (~100x) fewer bits than the raw cBF, matching the paper's claim of size reduction by two orders of magnitude at equal precision.

## Related tools

- **fimpera** (Produces the fimpera-wrapped per-read abundance queries against counting Bloom filters of varying sizes that are scored and plotted) — https://github.com/lrobidou/fimpera
- **Python** (Computes the sum-of-squared-error overestimation scores, tabulates size-vs-score pairs, and renders fig5_sse_tradeoff.png)

## Examples

```
./bin/fimpera_query index.idx query.fasta  # per-read abundances per filter size, then plot SSE overestimation score vs bits in Python
```

## Evaluation signals

- Recomputed overestimation score for fimpera at its reference operating point equals ~200000 with ~3.10x10^9 bits (~3.87x10^8 slots of 8 bits)
- Raw cBF at the same overestimation score (200000) requires ~330.9x10^9 bits, i.e. ~100x more space than fimpera
- Only overestimation errors contribute: squared differences with reported < original must be excluded from the accumulation
- The saved fig5_sse_tradeoff.png reproduces the reported Fig. 5 trade-off, with fimpera and cBF as separate series
- Bit sizes must be consistent with the 8-bits-per-slot filter configuration used for the fimpera index

## Limitations

- The comparison metric is an overestimation-only sum-of-squared-error score; underestimation errors are not captured by this trade-off analysis
- Results hold for the original (non-log) abundance storage; fimpera also supports log2-grouped abundance strategies which would alter the score semantics
- The fimpera implementation itself uses a counting Bloom filter; trade-off conclusions for other AMQ data structures are not established by this figure

## Evidence

- [abstract] Abstract finding on two-orders-of-magnitude size reduction: "Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision."
- [abstract] Abstract finding on false positive and abundance precision improvement: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [readme] README on fimpera as a wrapper around an abundance-capable AMQ: "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
- [readme] README on indexing command and filter sizing parameters: [evidence span withheld — anchor status: external_doc]
- [readme] README on log2 abundance grouping as an alternative strategy: [evidence span withheld — anchor status: not_found]
