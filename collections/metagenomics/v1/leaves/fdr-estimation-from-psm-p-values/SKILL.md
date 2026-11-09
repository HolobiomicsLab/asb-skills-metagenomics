---
name: fdr-estimation-from-psm-p-values
description: Use when tandem mass spectra must be searched against predicted RiPP
  structures and an FDR estimate is needed at a given p-value threshold via a target-decoy
  strategy.
when_to_use_negative:
- Spectra without paired genomics/metagenomics data — 99% of GNPS spectra cannot be
  searched, so no target-decoy FDR estimation is possible
- Cyclic peptides requiring blind modification searches, which MetaMiner does not
  perform due to inherent computational complexity
- 'General peptide ID where decoys should be sequence-only: here decoys must be shuffled
  BEFORE applying modifications, so the decoy database mirrors the target modification/mass
  profile rather than being a plain sequence scramble'
tools:
- name: MetaMiner
  role: Pipeline that constructs the target/decoy RiPP databases and computes FDR
    using the decoy database; available as a GNPS workflow and command line tool
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator
  role: Modified version used for searching spectra against the database of putative
    target/decoy RiPPs
- name: MS-DPR
  role: Computes the p-values of peptide-spectrum matches used for FDR estimation
- name: HMMer
  role: Detects modification enzymes on translated ORFs, which defines which PTMs
    are applied to target and decoy precursor peptides
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:e23993cc739506601c4fab65b500ee22f1e882b4a8a8cd0e5d49481649292e6d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# fdr-estimation-from-psm-p-values

## Summary

Target-decoy false discovery rate estimation for RiPP identifications in MetaMiner: decoy RiPP structures are built by randomly shuffling target precursor peptide sequences before applying the same post-translational modifications, and decoy-vs-target peptide-spectrum matches at MS-DPR p-value thresholds are used to estimate FDR.

## When to use

Use when tandem mass spectra must be searched against predicted RiPP structures and an FDR estimate is needed at a given p-value threshold via a target-decoy strategy.

## When NOT to use

- Spectra without paired genomics/metagenomics data — 99% of GNPS spectra cannot be searched, so no target-decoy FDR estimation is possible
- Cyclic peptides requiring blind modification searches, which MetaMiner does not perform due to inherent computational complexity
- General peptide ID where decoys should be sequence-only: here decoys must be shuffled BEFORE applying modifications, so the decoy database mirrors the target modification/mass profile rather than being a plain sequence scramble

## Inputs

- Target database of predicted RiPP structures (PTM-applied precursor peptides)
- Precursor peptide sequences (translated ORFs from motif-ORF/all-ORF construction)
- Tandem mass spectra (e.g., GNPS spectral datasets)
- Peptide-spectrum match p-values from MS-DPR

## Outputs

- Decoy RiPP database (shuffled precursor sequences with target modifications applied)
- FDR estimates at each p-value threshold (FDR = Ndecoy/Ntarget)
- RiPP identifications passing the 1% FDR threshold

## How to apply

First build the target RiPP structure database by applying the PTMs of modification enzymes detected with HMMer to their precursor peptides (translated ORFs from motif-ORF/all-ORF construction). Build the decoy database by randomly shuffling each target precursor peptide sequence BEFORE applying the same target modifications, so the decoys preserve the modification/mass profile while randomizing the sequence. Search spectra against both target and decoy RiPP databases using a modified version of Dereplicator, then compute peptide-spectrum match p-values with MS-DPR. Sweep a grid of p-value thresholds and, at each threshold, compute FDR = Ndecoy/Ntarget from target and decoy match counts; MetaMiner's reference operating point is a 1% FDR threshold for RiPP identification.

## Related tools

- **MetaMiner** (Pipeline that constructs the target/decoy RiPP databases and computes FDR using the decoy database; available as a GNPS workflow and command line tool) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator** (Modified version used for searching spectra against the database of putative target/decoy RiPPs)
- **MS-DPR** (Computes the p-values of peptide-spectrum matches used for FDR estimation)
- **HMMer** (Detects modification enzymes on translated ORFs, which defines which PTMs are applied to target and decoy precursor peptides)

## Evaluation signals

- Decoy database must be built by shuffling precursor sequences BEFORE applying target modifications — decoys carry the same modification/mass profile as targets
- FDR at each threshold equals Ndecoy/Ntarget for the corresponding p-value cutoff
- RiPP identifications are reported at a 1% FDR threshold (e.g., the 38 RiPPs identified from 10 million spectra)
- Positive-control sanity check: MetaMiner recovered all 18 known RiPPs in the positive control dataset under this FDR scheme
- Per the workflow, the target database, decoy database, and an FDR-vs-threshold table (fdr_vs_threshold.csv) should all be written to disk

## Limitations

- 99% of GNPS spectra are not accompanied by genomics/metagenomics data, making target-decoy searching impossible for them
- No blind modification searches are performed for cyclic peptides due to inherent computational complexity
- The motif-ORF precursor discovery strategy is not sensitive and misses peptides without sequence similarity to known RiPPs, which shrinks the target set the FDR is estimated against

## Evidence

- [discussion] creates a decoy database of RiPP precursor peptides by randomly shuffling each peptide in the target precursor database: "creates a decoy database of RiPP precursor peptides by randomly shuffling each peptide in the target precursor database"
- [readme] MetaMiner is currently available as a workflow at the Global Natural Products Social Molecular Networking (GNPS).: [evidence span withheld — anchor status: not_found]
