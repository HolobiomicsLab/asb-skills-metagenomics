---
name: parameter-sweep-evaluation
description: 'Use when selecting the k-mer length k for baseLess salient-k-mer models:
  sweep candidate k values against non-target lengths (30/50 kb) and target lengths
  (0.6/1.5/30 kb) and pick the k minimizing FPR.'
when_to_use_negative:
- The k-mer lengths are already fixed (e.g. using the pre-generated 16S k-mer detection
  networks shipped with baseLess) and no new k selection is required
- Empirical FPR on real read data is needed rather than the analytical Poisson estimate;
  this skill evaluates the analytical model only, not baseLess run_inference output
tools:
- name: baseLess
  role: The tool whose salient k-mer detection networks are being parameterized; the
    chosen k values (8, 9) feed into baseLess compile_model / k-mer network library
    design
  repo: https://github.com/cvdelannoy/baseLess
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:7b5e7742344c9c591c9c3557348313c0f9ef6599f2475625ecd0fd8a68f2da57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# parameter-sweep-evaluation

## Summary

Reproduce the analytical Poisson k-mer false-positive-rate sweep from Section 4.2 of the baseLess paper, which identified k=8 and k=9 as the k-mer sizes minimizing false positives for detecting salient subsequences of target sequences in raw MinION signal.

## When to use

Use when selecting the k-mer length k for baseLess salient-k-mer models: sweep candidate k values against non-target lengths (30/50 kb) and target lengths (0.6/1.5/30 kb) and pick the k minimizing FPR.

## When NOT to use

- The k-mer lengths are already fixed (e.g. using the pre-generated 16S k-mer detection networks shipped with baseLess) and no new k selection is required
- Empirical FPR on real read data is needed rather than the analytical Poisson estimate; this skill evaluates the analytical model only, not baseLess run_inference output

## Inputs

- Validation scripts from Zenodo deposit 10.4121/20261392 (baseLess paper supplementary)
- Candidate range of k-mer lengths k to sweep
- Non-target sequence lengths: 30 kb and 50 kb
- Target sequence lengths: 0.6 kb, 1.5 kb, and 30 kb

## Outputs

- fpr_table.csv: false-positive-rate table with rows = k values and columns = non-target/target length combinations
- Selected optimal k-mer sizes (expected: k=8 and k=9) for use in downstream baseLess k-mer network/model design

## How to apply

Retrieve the validation scripts accompanying baseLess from the Zenodo deposit 10.4121/20261392 (MIT licensed). Implement the analytical Poisson k-mer occurrence model of Section 4.2, computing the expected probability that non-target sequences of 30 kb and 50 kb spuriously contain the salient k-mers of target sequences of 0.6 kb, 1.5 kb, and 30 kb. Sweep k over the candidate range and evaluate the analytical FPR for every (non-target length, target length) combination, writing the full table (rows = k values, columns = length combinations) to fpr_table.csv. The sweep is validated only if the minimum FPR across all length combinations is attained at k=8 and k=9, matching the paper's reported result; both sizes were then carried forward into subsequent baseLess library design steps.

## Related tools

- **baseLess** (The tool whose salient k-mer detection networks are being parameterized; the chosen k values (8, 9) feed into baseLess compile_model / k-mer network library design) — https://github.com/cvdelannoy/baseLess

## Examples

```
python fpr_poisson_sweep.py --k-range 4:15 --non-target-lengths 30000,50000 --target-lengths 600,1500,30000 --out fpr_table.csv
```

## Evaluation signals

- The minimum FPR across all non-target/target length combinations is attained at exactly k=8 and k=9
- fpr_table.csv contains one row per swept k value and one column per (non-target length, target length) combination (30/50 kb vs 0.6/1.5/30 kb)
- Both k=8 and k=9 are the sizes carried forward into subsequent library design steps, matching the paper's reported result
- The Poisson model reproduces the analytical formulation used in Section 4.2, not an empirical read-level simulation

## Limitations

- The FPR is an analytical Poisson estimate over specified sequence lengths, not an empirical validation on real MinION reads; actual detection depends on the neural k-mer networks and the >66% k-mer fraction read-detection threshold
- The result was tuned in the context of 16S sequence detection; baseLess README notes the read_detection mode 'has been tested for and attuned to 16S sequence at the moment'

## Evidence

- [abstract] k-mer networks rationale: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [abstract] validation scripts location: "Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license."
- [readme] 16S tuning caveat: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
- [readme] read detection threshold: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
