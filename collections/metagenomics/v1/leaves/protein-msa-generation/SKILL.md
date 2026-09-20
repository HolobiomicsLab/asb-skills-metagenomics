---
name: protein-msa-generation
description: Use when starting from protein FASTA sequences that lack a supplied MSA
  and an MSA is required as input for DeepECA-style contact prediction, feature computation,
  or controlled noisy-MSA robustness tests.
when_to_use_negative:
- The DeepECA training/test datasets already ship with MSAs — download them from the
  Google Drive folder rather than regenerating them
- You need alignments with canonical DeepECA parameters for reproducibility but plan
  to use non-canonical E-value/coverage settings (those belong only to the explicit
  noisy-MSA robustness experiment)
- The downstream task does not consume an MSA at all (e.g. single-sequence structure
  prediction methods)
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: HHblits
  role: Iterative MSA generation against UniProt20_2016 with E-value 0.001 and 60%
    coverage; also used with relaxed settings (E-values 1 and 3, no -cov) to produce
    noisy MSAs for robustness testing
- name: DeepECA
  role: 'Downstream consumer: run_hhblits_local.py orchestrates MSA generation and
    multitask.py trains the Feature Added Model on the resulting MSAs'
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:0b1dac7c888084d31639328440f1937c3548809255e38828e0be35d30587f12c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# protein-msa-generation

## Summary

Generates multiple sequence alignments (MSAs) for protein domains with HHblits using DeepECA's canonical settings (three iterations, E-value 0.001, 60% coverage against UniProt20_2016), providing the core MSA input from which one-hot, PSSM, weighted-MSA entropy, and mutual-information features are computed for DNN-based contact prediction.

## When to use

Use when starting from protein FASTA sequences that lack a supplied MSA and an MSA is required as input for DeepECA-style contact prediction, feature computation, or controlled noisy-MSA robustness tests.

## When NOT to use

- The DeepECA training/test datasets already ship with MSAs — download them from the Google Drive folder rather than regenerating them
- You need alignments with canonical DeepECA parameters for reproducibility but plan to use non-canonical E-value/coverage settings (those belong only to the explicit noisy-MSA robustness experiment)
- The downstream task does not consume an MSA at all (e.g. single-sequence structure prediction methods)

## Inputs

- Protein amino acid sequences in FASTA format (placed in ./dataset/fasta for prediction)
- HHblits installation with a sequence library (UniProt20_2016) database
- Optionally, pre-computed MSAs and pre-trained models from the DeepECA Google Drive folder

## Outputs

- Per-protein multiple sequence alignments (MSAs) used to compute one-hot query, PSSM, weighted-MSA entropy, and mutual-information features
- MSA depth (number of homologous sequences) per protein for accuracy binning

## How to apply

Set up HHblits and edit the path to the hhblits binary in run_hhblits_local.py as instructed in the DeepECA README. Generate MSAs with three HHblits iterations against the UniProt20_2016 library using an E-value threshold of 0.001 and 60% coverage ('-cov 60'); the article also provides pre-computed MSAs in a Google Drive folder, so only generate MSAs for proteins not already covered. For deliberate robustness/noise experiments, deviate from the canonical settings by using E-values 1 and 3 and removing the '-cov' option to produce noisy MSAs. The resulting per-protein MSAs (depth varies widely across CASP11/CASP12 domains) are then cropped to 200 residues per sequence and subsampled to at most 30,000 sequences during training, and their depth is recorded because long-range accuracy gains are binned by MSA depth (top L/5, L/2, L).

## Related tools

- **HHblits** (Iterative MSA generation against UniProt20_2016 with E-value 0.001 and 60% coverage; also used with relaxed settings (E-values 1 and 3, no -cov) to produce noisy MSAs for robustness testing)
- **DeepECA** (Downstream consumer: run_hhblits_local.py orchestrates MSA generation and multitask.py trains the Feature Added Model on the resulting MSAs) — https://github.com/tomiilab/DeepECA

## Examples

```
python run_hhblits_local.py  # after editing the hhblits path; three iterations, E-value 0.001, -cov 60 on UniProt20_2016
```

## Evaluation signals

- HHblits was run with three iterations, E-value 0.001, and 60% coverage on the UniProt20_2016 library — verify parameters in the run logs
- Each target protein has a non-empty MSA, and the number of homologous sequences per MSA is recorded so proteins can be binned into MSA-depth strata
- Downstream validation: accuracy_comparison.csv shows the Feature Added Model matching or exceeding the Weighted MSA Model on long-range top L/5, L/2, and L on CASP11 (105 domains) and CASP12 (55 domains), especially for shallow MSAs
- During training, sequences are cropped to 200 residues and at most 30,000 MSA sequences are sampled per protein — check pipeline constraints were applied

## Limitations

- Shallow MSAs (few homologous sequences) limit achievable long-range accuracy; improving MSA construction and collecting sufficient evolutionary information from wider sequence spaces is explicitly left as future work
- Noisy MSAs (produced with E-values 1 and 3 without the -cov option) can contain metagenomic noise sequences, though DeepECA's sequence weighting can reduce such noise automatically
- The hhblits binary path must be manually edited in run_hhblits_local.py; training is reported to take 2–3 days on an NVIDIA TITAN GTX GPU and GPU memory limits the downstream network to 60 layers

## Evidence

- [discussion] We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [discussion] The threshold E-value was set to 0.001 on the UniProt20_2016 library: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
- [intro] We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs: "We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs"
- [readme] You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately.: "You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately."
