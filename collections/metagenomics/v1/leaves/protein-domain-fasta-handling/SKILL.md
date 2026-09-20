---
name: protein-domain-fasta-handling
description: Use when the input is a raw target protein amino-acid sequence (or set
  of sequences) that must be converted to FASTA and staged so HHblits-generated MSAs
  and covariance features can be computed for contact prediction.
when_to_use_negative:
- The input already consists of MSAs or precomputed covariance/precision matrices
  rather than raw FASTA sequences (e.g. ResPRE-style precision-matrix inputs)
- The task requires contact prediction without any MSA at all, or for proteins so
  shallow that HHblits with E-value 0.001 and 60% coverage yields no usable alignment
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: DeepECA
  role: Framework consuming the staged FASTA inputs; runs HHblits-based MSA generation,
    covariance feature computation, and DNN contact/secondary-structure prediction
  repo: github.com/tomiilab/DeepECA
- name: HHblits
  role: Builds one MSA per target FASTA (three iterations, E-value 0.001, '-cov 60')
    against the UniProt20_2016 library
- name: PSICOV
  role: Its frequency formula is used to compute observed single-site and pairwise
    amino-acid frequencies from each MSA for covariance features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:7626a1122a63edada1944675f2af0783307d32ff3a1482d95c1cd2734ec0db8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepECA
  - HHblits
  - PSICOV
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# protein-domain-fasta-handling

## Summary

Prepare target protein amino-acid sequences as FASTA files and stage them in the DeepECA workspace so HHblits can build one MSA per protein, which is then converted to 441 covariance-matrix input features for end-to-end contact prediction.

## When to use

Use when the input is a raw target protein amino-acid sequence (or set of sequences) that must be converted to FASTA and staged so HHblits-generated MSAs and covariance features can be computed for contact prediction.

## When NOT to use

- The input already consists of MSAs or precomputed covariance/precision matrices rather than raw FASTA sequences (e.g. ResPRE-style precision-matrix inputs)
- The task requires contact prediction without any MSA at all, or for proteins so shallow that HHblits with E-value 0.001 and 60% coverage yields no usable alignment

## Inputs

- Target protein amino-acid sequences in FASTA format (placed in ./dataset/fasta)
- UniProt20_2016 sequence library for HHblits MSA construction
- Pre-trained DeepECA models and datasets from the Google Drive folder (for prediction)

## Outputs

- One HHblits-generated MSA per target protein
- Per-protein 441-channel tensor of L-by-L covariance matrices derived from PSICOV frequencies
- Predicted contacts (./contact_pred) and secondary structures (./ss_pred) from the multitask model

## How to apply

Place all target FASTA files into ./dataset/fasta so the DeepECA pipeline (pipeline.py) picks them up. Set up HHblits locally and edit the hhblits path in run_hhblits_local.py, then build one MSA per target using three iterations, an E-value threshold of 0.001 against the UniProt20_2016 library, and the '-cov 60' coverage option. For each MSA, compute single-site and pairwise amino-acid frequencies with the PSICOV frequency formula and assemble the full set of 441 L-by-L covariance matrices (for protein length L) as the network input. Note that training-time preprocessing crops 200 residues and samples up to 30,000 MSA sequences; contacts are defined as Cβ–Cβ distances under 8 Å (Cα for glycine).

## Related tools

- **DeepECA** (Framework consuming the staged FASTA inputs; runs HHblits-based MSA generation, covariance feature computation, and DNN contact/secondary-structure prediction) — github.com/tomiilab/DeepECA
- **HHblits** (Builds one MSA per target FASTA (three iterations, E-value 0.001, '-cov 60') against the UniProt20_2016 library)
- **PSICOV** (Its frequency formula is used to compute observed single-site and pairwise amino-acid frequencies from each MSA for covariance features)

## Examples

```
cp target_*.fasta ./dataset/fasta && python run_hhblits_local.py && python pipeline.py
```

## Evaluation signals

- Each target FASTA in ./dataset/fasta yields a successfully generated HHblits MSA
- The covariance tensor for each protein has the expected 441 channels and L-by-L dimensions equal to the protein length L
- Prediction results are written to ./contact_pred and ./ss_pred when running the multitask pipeline
- For training, cropped sequences are 200 residues and MSA sampling caps at 30,000 sequences
- Contacts labeled during evaluation satisfy the Cβ < 8 Å distance definition (Cα for glycine)

## Limitations

- Shallow MSAs limit accuracy; improving MSA construction and collecting sufficient evolutionary information for shallow MSAs is left as future work
- Proteins longer than 700 residues or shorter than 25 residues were excluded during dataset curation, so extreme-length targets may be problematic
- Training the models from scratch requires 2–3 days on an NVIDIA TITAN GTX GPU, and 60 network layers were used only because of GPU memory limitations

## Evidence

- [readme] When you want to use your original fasta file, put all your fasta files into ./dataset/fasta .: "When you want to use your original fasta file, put all your fasta files into ./dataset/fasta ."
- [readme] You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately.: "You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately."
- [discussion] We prepared MSAs for proteins in both our original and test datasets using HHblits with three iterations: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [discussion] The threshold E-value was set to 0.001 on the UniProt20_2016 library: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
- [intro] The recently reported DeepCov, which is a conceptually similar method to ours uses a covariance matrix calculated from MSA for input features for DNN.: "The recently reported DeepCov [21], which is a conceptually similar method to ours uses a covariance matrix calculated from MSA for input features for DNN."
