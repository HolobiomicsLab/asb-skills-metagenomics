---
name: false-positive-rate-analysis
description: 'Use when choosing k-mer lengths for baseLess''s composite k-mer detection
  networks: run the Section 4.2 analytical Poisson FPR sweep over candidate k values
  against non-target (30/50 kb) and target (0.6/1.5/30 kb) length combinations.'
when_to_use_negative:
- The salient k-mer list is already fixed and you only need to compile a model — use
  `baseLess compile_model --model-type abundance` or `--model-type read_detection`
  instead of re-running the k-selection simulation
- You need an empirical false-positive rate measured on real MinION fast5 reads rather
  than the analytical Poisson estimate over assumed sequence lengths
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: baseLess
  role: k-mer detection tool whose composite network of small k-mer-detecting neural
    networks is parameterized by the k values (k=8, k=9) selected via this FPR simulation
  repo: https://github.com/cvdelannoy/baseLess
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:258110b58c45665a796785a26d7a950442e2d5323b8c0c150226ff86a4e1aca0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# false-positive-rate-analysis

## Summary

Analytical false-positive-rate simulation from Section 4.2 of the baseLess paper: a Poisson k-mer occurrence model that sweeps k-mer length k over candidate sizes to find which k minimize spurious k-mer hits in non-target sequences. The paper's result (k=8 and k=9) feeds baseLess's composite k-mer detection network library.

## When to use

Use when choosing k-mer lengths for baseLess's composite k-mer detection networks: run the Section 4.2 analytical Poisson FPR sweep over candidate k values against non-target (30/50 kb) and target (0.6/1.5/30 kb) length combinations.

## When NOT to use

- The salient k-mer list is already fixed and you only need to compile a model — use `baseLess compile_model --model-type abundance` or `--model-type read_detection` instead of re-running the k-selection simulation
- You need an empirical false-positive rate measured on real MinION fast5 reads rather than the analytical Poisson estimate over assumed sequence lengths

## Inputs

- Candidate k-mer sizes k to sweep
- Non-target sequence lengths: 30 kb and 50 kb
- Target sequence lengths: 0.6 kb, 1.5 kb, and 30 kb
- Validation scripts from Zenodo deposit doi:10.4121/20261392 (MIT license)

## Outputs

- fpr_table.csv (rows = k values, columns = non-target/target length combinations)
- Optimal k-mer lengths minimizing FPR (paper reports k=8 and k=9) for use in the k-mer detection network library

## How to apply

Retrieve the validation scripts from the Zenodo deposit 10.4121/20261392 that accompany the baseLess paper, then implement the analytical Poisson k-mer occurrence model used in Section 4.2: compute the expected probability that non-target sequences of length 30 kb and 50 kb spuriously contain the salient k-mers of target sequences of length 0.6 kb, 1.5 kb, and 30 kb. Sweep the k-mer size k over the candidate range and evaluate the analytical FPR for every combination of non-target length and target length. Write the full FPR table (rows = k values, columns = non-target/target length combinations) to fpr_table.csv. Validate that the minimum FPR across all length combinations is attained at k=8 and k=9, matching the paper's reported result; both sizes were then included in subsequent library design steps for the k-mer detection networks.

## Related tools

- **baseLess** (k-mer detection tool whose composite network of small k-mer-detecting neural networks is parameterized by the k values (k=8, k=9) selected via this FPR simulation) — https://github.com/cvdelannoy/baseLess

## Evaluation signals

- The minimum analytical FPR across all non-target (30/50 kb) × target (0.6/1.5/30 kb) length combinations is attained at k=8 and k=9
- fpr_table.csv has one row per swept k value and one column per non-target/target length combination, with FPR values in (0,1)
- Re-running with the Zenodo validation scripts (doi:10.4121/20261392) reproduces the paper's Section 4.2 FPR table
- The selected k values (8 and 9) are carried forward into the k-mer detection network library design steps, mirroring the paper

## Limitations

- This is an analytical Poisson model over assumed sequence lengths, not an empirical measurement on sequenced data; the k=8/k=9 optimum was validated only for target lengths 0.6/1.5/30 kb versus non-target lengths 30/50 kb, so other length regimes may shift the optimum
- Pre-generated k-mer detection networks are attuned to 16S detection; the README notes the authors plan to further expand the number of available k-mer networks
- The related read-detection filter (>66% k-mer fraction per read) was tested for and attuned to 16S sequence, so downstream FPR behavior may differ for other targets

## Evidence

- [abstract] Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license.: "Used validation data and scripts can be found at https://doi.org/10.4121/20261392, under an MIT license."
- [results] The FPR simulation identified k=8 and k=9 as the k-mer lengths returning good (minimizing) false positive rates, and both sizes were included in subsequent library design steps.: [evidence span withheld — anchor status: not_found]
- [readme] The neural network it uses is a composite of many smaller networks, each detecting a different k-mer: "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer"
- [readme] baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection.: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
