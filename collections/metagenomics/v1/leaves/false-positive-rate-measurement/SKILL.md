---
name: false-positive-rate-measurement
description: Use when benchmarking how a counting AMQ (e.g., a counting Bloom filter
  built with fimpera) calls k-mers/reads incorrectly, or when quantifying the FPR
  gain or space savings from wrapping a cBF with fimpera at a given z threshold.
when_to_use_negative:
- The structure does not support abundance queries — fimpera requires an AMQ with
  counts (a plain set-membership Bloom filter cannot be wrapped)
- No ground-truth k-mer abundances are available for the query dataset, making FPR
  computation impossible
- The goal is presence/absence indexing only with no counts and no abundance-based
  false positive analysis
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Builds counting Bloom filter indexes and performs z-thresholded abundance
    queries used for FPR measurement
  repo: https://github.com/lrobidou/fimpera
- name: counting Bloom filter
  role: Baseline raw cBF against which the fimpera-wrapped false positive rate is
    compared at equal space
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:e28291e162ab4f5cd97bb995e709c8035adcef26c356f870cb216d6eee234b1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - counting Bloom filter
  - fimpera
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# false-positive-rate-measurement

## Summary

Protocol for measuring the false positive rate (FPR) of Approximate Membership Query data structures with counts — raw counting Bloom filters versus fimpera-wrapped filters — by comparing query calls against ground-truth k-mer abundances across abundance thresholds z.

## When to use

Use when benchmarking how a counting AMQ (e.g., a counting Bloom filter built with fimpera) calls k-mers/reads incorrectly, or when quantifying the FPR gain or space savings from wrapping a cBF with fimpera at a given z threshold.

## When NOT to use

- The structure does not support abundance queries — fimpera requires an AMQ with counts (a plain set-membership Bloom filter cannot be wrapped)
- No ground-truth k-mer abundances are available for the query dataset, making FPR computation impossible
- The goal is presence/absence indexing only with no counts and no abundance-based false positive analysis

## Inputs

- Reference fastq file for index construction (e.g., TARA accession ERR1726642)
- Query fastq file for benchmarking (e.g., TARA accession ERR4691696)
- Ground-truth k-mer abundance list (e.g., KMC output: one k-mer per line, tab, abundance)
- Fimpera index parameters: k=31, b=5 bits per bucket, z threshold sweep 0–20, log2 abundance transformation

## Outputs

- fpr_table.csv: FPR per z value (21 rows, z=0..20), reproducing Table 2 columns
- fig4_curve.csv: filter size, FPR, memory in bits, and method (fimpera vs baseline cBF) per data point
- Per-read / per-k-mer query call outputs from fimpera_query compared against ground truth

## How to apply

Build a counting Bloom filter index from a reference fastq (e.g., TARA accessions such as ERR1726642) using fimpera with k=31, b=5 bits per bucket, and log2-transformed abundances, then query it with a held-out fastq (e.g., ERR4691696) at abundance thresholds z from 0 to 20, recording called k-mers/reads. Compute the FPR at each z by comparing fimpera's calls against ground-truth k-mer abundances; expect ~25% FPR for the raw cBF (designed for 25% false positive calls) dropping to ~0.4–0.56% at the recommended default z=3 with identical space, and near-100% FPR at z=20. For the space-versus-FPR trade-off (Fig. 4), sweep counting Bloom filter sizes indexing k-mers with original (non-log) abundances, build matched plain cBF baselines, and measure FPR and memory footprint in bits per filter size; expect fimpera to reach 2% FPR at ~2.98×10^9 bits versus ~65.5×10^9 bits (~21× more) without fimpera, and 0.02% FPR at ~7.25×10^9 bits versus >360×10^9 bits for the baseline. Assemble results into CSV tables (e.g., fpr_table.csv with 21 rows for z=0..20, and fig4_curve.csv with filter size, FPR, memory in bits, and method columns).

## Related tools

- **fimpera** (Builds counting Bloom filter indexes and performs z-thresholded abundance queries used for FPR measurement) — https://github.com/lrobidou/fimpera
- **counting Bloom filter** (Baseline raw cBF against which the fimpera-wrapped false positive rate is compared at equal space)

## Examples

```
./bin/fimpera_index kmc_abundances.txt index.idx 1000000 -k 31 -b 5 -z 3 ; ./bin/fimpera_query index.idx query.fastq
```

## Evaluation signals

- fpr_table.csv exists and contains exactly 21 rows (z=0..20)
- FPR at z=3 matches the reported ~0.4% / 0.56% values from Table 2, and FPR at z=20 approaches 100%
- Raw cBF query FPR is ~25%, matching the filter's design target for false positive calls
- fig4_curve.csv shows fimpera reaching 2% FPR at ~2.98×10^9 bits and 0.02% FPR at ~7.25×10^9 bits, versus ~65.5×10^9 and >360×10^9 bits for the baseline (~21×–50× space reduction)
- Fimpera-wrapped and raw cBF indexes occupy the same number of bits when comparing FPR at equal space

## Limitations

- The FPR reported at z=20 is near 100%, so threshold choice dominates the trade-off and must be swept rather than fixed
- Fimpera requires a file of k-mer abundances (e.g., KMC output) for indexing; it cannot index directly from raw reads without a counting step
- The paper-companion reproduction deposit instructions referenced in the README are marked TODO, so workflow reproduction relies on the separate reproduce-commands deposit

## Evidence

- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [abstract] Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision.: "Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision."
- [readme] The fimpera implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not).: "The `fimpera` implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not)."
- [readme] fimpera needs a file containing the abundance of each k-mers to index them: [evidence span withheld — anchor status: external_doc]
- [readme] ./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3: [evidence span withheld — anchor status: external_doc]
