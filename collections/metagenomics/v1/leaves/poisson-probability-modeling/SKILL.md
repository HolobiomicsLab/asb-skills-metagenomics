---
name: poisson-probability-modeling
description: Decision support for choosing the k-mer length k for a k-mer-detection
  model (e.g. baseLess composite networks) by computing expected false positive rates
  of salient k-mers occurring by chance in non-target sequences.
when_to_use_negative:
- Target sequences are so long that k-mers occur by chance with negligible Poisson
  probability — the simulation would not change the k choice.
- The salient k-mer set is already fixed (e.g. pre-generated 16S k-mer networks from
  `baseLess update_16s_db`) and no re-selection of k is needed.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: baseLess
  role: Target-detection tool whose composite of per-k-mer networks motivates the
    FPR simulation; the optimal k values feed its compile_model steps
  repo: https://github.com/cvdelannoy/baseLess
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:8e15d2ff85cf53b56a20eb7a1772b7864255952ac8d181edbe5b1c656f78c339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# poisson-probability-modeling

## Summary

Analytical Poisson k-mer occurrence modeling used in the baseLess paper (Section 4.2) to predict the probability that non-target sequences spuriously contain salient k-mers of a target sequence, and to select the k-mer sizes (k=8, k=9) that minimize the false positive rate for library design.

## When to use

Decision support for choosing the k-mer length k for a k-mer-detection model (e.g. baseLess composite networks) by computing expected false positive rates of salient k-mers occurring by chance in non-target sequences.

## When NOT to use

- Target sequences are so long that k-mers occur by chance with negligible Poisson probability — the simulation would not change the k choice.
- The salient k-mer set is already fixed (e.g. pre-generated 16S k-mer networks from `baseLess update_16s_db`) and no re-selection of k is needed.

## Inputs

- Candidate range of k-mer sizes k
- Non-target (background) sequence lengths: 30 kb and 50 kb
- Target sequence lengths: 0.6 kb, 1.5 kb, and 30 kb
- Validation scripts from Zenodo deposit 10.4121/20261392

## Outputs

- Analytical false-positive-rate values for every (k, non-target length, target length) combination
- fpr_table.csv (rows = k values, columns = non-target/target length combinations)
- Selected optimal k-mer sizes (expected: k=8 and k=9) to feed into subsequent k-mer model compilation

## How to apply

Retrieve the validation scripts from the Zenodo deposit 10.4121/20261392 accompanying the baseLess paper and implement the analytical Poisson k-mer occurrence model from Section 4.2. For each candidate k-mer size, compute the expected probability that non-target sequences of length 30 kb and 50 kb spuriously contain the salient k-mers of target sequences of length 0.6 kb, 1.5 kb, and 30 kb, sweeping k over the candidate range and evaluating the FPR for every non-target/target length combination. Write the full FPR table (rows = k values, columns = length combinations) to fpr_table.csv. This step matters because baseLess's composite neural network combines many smaller networks, each detecting one k-mer, so the k-mer set directly determines how easily a non-target read is misclassified; the read_detection classifier treats a read as target when it contains >66% of the total salient k-mers, making FPR-minimizing k choices critical. Validate by checking that the minimum FPR across all length combinations is attained at k=8 and k=9, matching the paper's result, and carry both sizes into subsequent library design.

## Related tools

- **baseLess** (Target-detection tool whose composite of per-k-mer networks motivates the FPR simulation; the optimal k values feed its compile_model steps) — https://github.com/cvdelannoy/baseLess

## Evaluation signals

- fpr_table.csv contains one FPR value per (k, non-target length, target length) combination with no missing cells.
- The minimum FPR across all 30 kb/50 kb vs 0.6/1.5/30 kb length combinations is attained at k=8 and k=9.
- Both k=8 and k=9 are retained in the subsequent library design / compile_model steps, matching the paper's reported result.
- The Poisson model is implemented analytically (expected occurrence probabilities), not by Monte-Carlo sampling, consistent with the Section 4.2 approach.

## Limitations

- The analytical FPR depends on the assumed sequence lengths (30/50 kb non-target, 0.6/1.5/30 kb target); results do not automatically transfer to very different length regimes.
- The read_detection classifier threshold (>66% of total k-mers, by default) has been tested for and attuned to 16S sequences, so k selection via this FPR model may not generalize to non-16S targets.

## Evidence

- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [abstract] Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license.: "Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license."
- [intro] The neural network it uses is a composite of many smaller networks, each detecting a different k-mer, thus reconfiguration of baseLess to detect a different sequence amounts to combining different: "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer, thus reconfiguration of baseLess to detect a different sequence amounts to combining different"
- [readme] If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species.: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
