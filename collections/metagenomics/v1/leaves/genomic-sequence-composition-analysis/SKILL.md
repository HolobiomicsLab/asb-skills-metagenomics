---
name: genomic-sequence-composition-analysis
description: 'Use when designing k-mer-based sequence detectors and you need to choose
  the k-mer length k: sweep k over a candidate range and compute the analytical false-positive
  rate of spurious k-mer occurrence for given target/non-target sequence lengths.'
when_to_use_negative:
- You already have a trained baseLess model or pre-generated 16S k-mer networks —
  no k-size sweep is needed
- Input is raw MinION fast5 signal data for inference (use baseLess run_inference,
  not composition analysis)
- The goal is abundance estimation against whole genomes rather than read-level salient
  k-mer selection
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: baseLess
  role: The composite per-k-mer neural-network detector whose salient k-mer sizes
    are chosen by this FPR simulation; validation scripts accompany the Zenodo deposit
    10.4121/20261392
  repo: https://github.com/cvdelannoy/baseLess
- name: MinION
  role: The sequencing instrument producing the raw electrical signal in which the
    selected k-mers are detected without basecalling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:b63427340d206f006539b4678094a02404db775dd644d2fb058e543b7984cda0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - MinION
  - baseLess
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# genomic-sequence-composition-analysis

## Summary

Analyzes k-mer composition of target and non-target sequences via an analytical Poisson occurrence model to select the k-mer lengths (k) whose salient k-mers minimize false-positive detection, as done in baseLess Section 4.2 to select k=8 and k=9.

## When to use

Use when designing k-mer-based sequence detectors and you need to choose the k-mer length k: sweep k over a candidate range and compute the analytical false-positive rate of spurious k-mer occurrence for given target/non-target sequence lengths.

## When NOT to use

- You already have a trained baseLess model or pre-generated 16S k-mer networks — no k-size sweep is needed
- Input is raw MinION fast5 signal data for inference (use baseLess run_inference, not composition analysis)
- The goal is abundance estimation against whole genomes rather than read-level salient k-mer selection

## Inputs

- Target sequence lengths (e.g. 0.6 kb, 1.5 kb, 30 kb) or FASTA of target sequences
- Non-target sequence lengths (e.g. 30 kb, 50 kb) or FASTA of background sequences
- Candidate range of k-mer sizes k to sweep
- Salient k-mer lists (e.g. from 16S targets) used for library design

## Outputs

- Full FPR table (rows = k values, columns = non-target/target length combinations), e.g. fpr_table.csv
- Selected optimal k values (k=8 and k=9 in the baseLess Section 4.2 simulation)
- k-mer list for downstream network/library design (txt list, 1 line per k-mer)

## How to apply

Implement the analytical Poisson k-mer occurrence model from Section 4.2: for each candidate k, compute the expected probability that a non-target sequence (e.g. 30 kb and 50 kb in the baseLess study) spuriously contains the salient k-mers drawn from target sequences of length 0.6 kb, 1.5 kb, and 30 kb. Sweep k over the candidate range and evaluate the FPR for every combination of non-target and target length. Write the full FPR table with rows as k values and columns as length combinations (e.g. to fpr_table.csv) and inspect which k values attain the minimum. In the baseLess workflow, the k-mers selected this way become the composite of small per-k-mer neural networks that detect fixed-size subsequences directly from the MinION electrical signal; k=8 and k=9 minimized the false-positive rate and were carried into subsequent library design. The chosen k-mers should be 'salient' — distinctive in the target context (e.g. 16S) — since baseLess ships pre-generated networks only for such salient k-mers.

## Related tools

- **baseLess** (The composite per-k-mer neural-network detector whose salient k-mer sizes are chosen by this FPR simulation; validation scripts accompany the Zenodo deposit 10.4121/20261392) — https://github.com/cvdelannoy/baseLess
- **MinION** (The sequencing instrument producing the raw electrical signal in which the selected k-mers are detected without basecalling)

## Examples

```
python fpr_simulation.py --non-target-lengths 30000 50000 --target-lengths 600 1500 30000 --k-sweep 4-16 --out fpr_table.csv
```

## Evaluation signals

- The minimum FPR across all non-target/target length combinations is attained at k=8 and k=9, matching the paper's Section 4.2 result
- fpr_table.csv contains one row per candidate k and one column per non-target/target length combination (30/50 kb vs 0.6/1.5/30 kb)
- FPR varies non-trivially with k for every length combination (model is sensitive to k, not constant)
- Selected k-mers fall within the pre-generated network set scope: k-mers 'particularly salient in the context of 16S detection'
- Re-running the Poisson model with identical lengths and k sweep reproduces the same optimal k values

## Limitations

- The analytical Poisson model evaluates spurious occurrence in random non-target sequences; empirical FPR on real reads may differ
- Pre-generated k-mer networks are attuned to 16S sequence; extending to other targets requires generating networks from training reads corrected with tombo resquiggle
- Read-level detection relies on a threshold (a read is called target if it contains >66% of total k-mers by default), which the k selection does not directly optimize

## Evidence

- [abstract] Composite per-k-mer networks: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [readme] Salient k-mer network scope: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
- [readme] Read-level k-mer fraction threshold: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
- [other] Optimal k values from FPR simulation: [evidence span withheld — anchor status: not_found]
