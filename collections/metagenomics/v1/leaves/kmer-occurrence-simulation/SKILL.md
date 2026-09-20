---
name: kmer-occurrence-simulation
description: 'Decision support for choosing k-mer lengths for baseLess salient k-mer
  selection: given target sequence lengths (e.g. 0.6, 1.5, 30 kb) and non-target sequence
  lengths (30/50 kb), simulate which k values minimize spurious k-mer hits.'
when_to_use_negative:
- You need empirical false-positive rates from actual basecalled or raw MinION reads
  rather than an analytical Poisson occurrence model
- Your sequences are 16S amplicons where baseLess's pre-generated 16S k-mer networks
  already apply and no k re-selection is needed
- The target/non-target length regime differs from the 0.6/1.5/30 kb target and 30/50
  kb non-target setup, unless the analytical model is re-parameterized for those lengths
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: baseLess
  role: Tool whose salient k-mer detection networks this simulation parameterizes;
    validation scripts accompany the baseLess deposit
  repo: https://github.com/cvdelannoy/baseLess
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:2843962dd61700b672715510ab4518d0a765094301fdeaa89409d6f32637df86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# kmer-occurrence-simulation

## Summary

Analytical Poisson-based simulation of spurious k-mer occurrence (Section 4.2 of the baseLess paper) used to identify which k-mer lengths k minimize the false-positive rate when detecting target subsequences in non-target reads, so that optimal k values feed into k-mer detection network design.

## When to use

Decision support for choosing k-mer lengths for baseLess salient k-mer selection: given target sequence lengths (e.g. 0.6, 1.5, 30 kb) and non-target sequence lengths (30/50 kb), simulate which k values minimize spurious k-mer hits.

## When NOT to use

- You need empirical false-positive rates from actual basecalled or raw MinION reads rather than an analytical Poisson occurrence model
- Your sequences are 16S amplicons where baseLess's pre-generated 16S k-mer networks already apply and no k re-selection is needed
- The target/non-target length regime differs from the 0.6/1.5/30 kb target and 30/50 kb non-target setup, unless the analytical model is re-parameterized for those lengths

## Inputs

- Candidate range of k-mer sizes k to sweep
- Non-target sequence lengths: 30 kb and 50 kb
- Target sequence lengths: 0.6 kb, 1.5 kb, and 30 kb
- Validation scripts from Zenodo deposit 10.4121/20261392 (baseLess paper, Section 4.2 Poisson model)

## Outputs

- fpr_table.csv: full FPR table with rows = k values, columns = non-target/target length combinations
- Identified optimal k-mer sizes (expected: k=8 and k=9) for downstream library design steps

## How to apply

Retrieve the validation scripts from the Zenodo deposit 10.4121/20261392 accompanying the baseLess paper (baseLess uses an array of small neural networks, each detecting a fixed-size subsequence, so the k choice governs detection specificity). Implement the analytical Poisson k-mer occurrence model: for each k, compute the expected probability that non-target sequences of length 30 kb and 50 kb spuriously contain the salient k-mers of target sequences of length 0.6, 1.5, and 30 kb. Sweep k over the candidate range and evaluate the analytical FPR for every combination of non-target length and target length. Write the full FPR table (rows = k values, columns = non-target/target length combinations) to fpr_table.csv. Validation: the minimum FPR across all length combinations must be attained at k=8 and k=9, matching the paper's reported result, and both k=8 and k=9 should be carried forward into subsequent library design (k-mer detection network) steps.

## Related tools

- **baseLess** (Tool whose salient k-mer detection networks this simulation parameterizes; validation scripts accompany the baseLess deposit) — https://github.com/cvdelannoy/baseLess

## Examples

```
from scipy.stats import poisson; lam = (L_non_target - k + 1) * (1/4)**k; p_fp = 1 - poisson.pmf(0, lam)  # per (k, L_non_target, L_target); sweep k and write to fpr_table.csv
```

## Evaluation signals

- Minimum FPR across all non-target/target length combinations is attained at k=8 and k=9
- Both k=8 and k=9 are included in subsequent library design steps, matching the paper's workflow
- fpr_table.csv contains a complete row per swept k value and a column per non-target/target length combination (30/50 kb vs 0.6/1.5/30 kb)
- Analytical Poisson probabilities are consistent between the independent reimplementation and the Zenodo 10.4121/20261392 validation scripts

## Limitations

- The model is analytical (Poisson k-mer occurrence), not an empirical simulation over real ONT signal or basecalled reads
- Result is attuned to the paper's specific length combinations; other target/non-target length regimes may shift the optimal k away from 8 and 9
- baseLess's pre-generated k-mer detection networks are noted as particularly salient for 16S, so k selection via this simulation may not transfer to other sequence contexts without additional validation

## Evidence

- [abstract] Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license.: "Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license."
- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [readme] The neural network it uses is a composite of many smaller networks, each detecting a different k-mer, thus reconfiguration of baseLess to detect a different sequence amounts to combining different: "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer, thus reconfiguration of baseLess to detect a different sequence amounts to combining different"
- [methods] If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species.: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
