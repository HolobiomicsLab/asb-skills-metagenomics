---
name: abundance-threshold-parameter-sweep
description: Use when you need to quantify how fimpera's threshold parameter z controls
  false positive calls on a counting Bloom filter, e.g. verifying the reported ~0.4%/0.56%
  FPR at z=3 and near-100% FPR at z=20.
when_to_use_negative:
- When only a single fixed threshold query is needed (e.g. the recommended z=3) rather
  than characterizing the whole z trade-off curve
- When the index is not an AMQ supporting abundance queries — fimpera requires abundance-supporting
  structures such as counting Bloom filters
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Builds the cBF index (fimpera_index) and runs queries at each abundance threshold
    z (fimpera_query) across the sweep
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:9fcd882bdafd48664d5d8ebf853051a027535924977fa02b0e1172e2b4c44620
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - fimpera
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# abundance-threshold-parameter-sweep

## Summary

Sweeps fimpera's abundance threshold z (0–20) on a counting Bloom filter index to map the trade-off between false positive rate and retained k-mer calls, reproducing the paper's Table 2.

## When to use

Use when you need to quantify how fimpera's threshold parameter z controls false positive calls on a counting Bloom filter, e.g. verifying the reported ~0.4%/0.56% FPR at z=3 and near-100% FPR at z=20.

## When NOT to use

- When only a single fixed threshold query is needed (e.g. the recommended z=3) rather than characterizing the whole z trade-off curve
- When the index is not an AMQ supporting abundance queries — fimpera requires abundance-supporting structures such as counting Bloom filters

## Inputs

- Counting Bloom filter index built by fimpera_index (k=31, b=5, log2 abundance binning)
- KMC output text file of k-mer / abundance pairs for indexing
- Query fastq (ERR4691696)
- Ground-truth k-mer abundances for FPR computation

## Outputs

- Called k-mers/reads per read for each z value (fimpera_query output)
- fpr_table.csv with 21 rows (z=0..20) of false positive rates reproducing Table 2

## How to apply

Install fimpera from the lrobidou/fimpera deposit and build a counting Bloom filter index from a fastq (here ERR1726642) with k=31, b=5 bits per bucket, and log2-transformed abundances supplied as a KMC-style k-mer/abundance text file. Query the index with a second fastq (ERR4691696) once per threshold z from 0 to 20, recording called k-mers and reads at each setting. Compare each query call against ground-truth k-mer abundances to compute a false positive rate per z. Assemble the 21 FPR values (z=0..20) into a table mirroring Table 2; the raw cBF alone yields a 25% FPR while fimpera-wrapped queries drop to ~0.56% at the recommended z=3, with the same memory footprint.

## Related tools

- **fimpera** (Builds the cBF index (fimpera_index) and runs queries at each abundance threshold z (fimpera_query) across the sweep) — https://github.com/lrobidou/fimpera

## Examples

```
./bin/fimpera_index kmers_with_abundances.txt index.idx 1000000 -k 31 -b 5 -z 3 ; ./bin/fimpera_query index.idx ERR4691696.fastq
```

## Evaluation signals

- fpr_table.csv exists and contains exactly 21 rows corresponding to z = 0..20
- FPR at z=3 falls in the reported range (~0.4%/0.56%)
- FPR at z=20 approaches 100%
- Querying the raw cBF without fimpera reproduces the designed 25% FPR, confirming the space budget is unchanged

## Limitations

- Near-zero-threshold settings (low z) retain high FPR, while very high z (e.g. 20) can reject nearly all calls (near-100% FPR observed in the sweep)
- The fimpera implementation as provided is specific to counting Bloom filters; adaptation to other AMQs requires templating work (fimpera-lib headers)
- fimpera needs a file of one k-mer per line with its abundance (KMC format); other counter output must be converted

## Evidence

- [results] recommended z=3 setting drops cBF FPR to 0.56%: [evidence span withheld — anchor status: not_found]
- [abstract] fimpera reduces cBF FPR by two orders of magnitude: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude"
- [readme] indexing command with k, b, z parameters: [evidence span withheld — anchor status: external_doc]
- [readme] fimpera requires k-mer abundance file for indexing: [evidence span withheld — anchor status: external_doc]
