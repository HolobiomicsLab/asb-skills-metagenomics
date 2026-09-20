---
name: ripp-target-decoy-database-construction
description: Use when searching tandem mass spectra against predicted RiPP structures
  and a calibrated FDR (e.g., a 1% identification threshold) must be estimated via
  a target-decoy strategy.
when_to_use_negative:
- The spectra are not accompanied by paired genomics/metagenomics data (per the article,
  the other 99% of GNPS spectra cannot be searched).
- You need blind modification searches for cyclic peptides — MetaMiner does not perform
  these due to inherent computational complexity.
- Your precursor peptides have no sequence similarity to known RiPPs and you rely
  solely on the motif-ORF strategy, which misses such peptides (all-ORF mode is needed
  instead).
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MetaMiner
  role: Command-line tool / GNPS workflow that constructs the target and decoy RiPP
    precursor databases and computes FDR from the decoy database
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator
  role: Modified version used to search spectra against the database of putative target/decoy
    RiPPs
- name: MS-DPR
  role: Computes p-values of the peptide-spectrum matches used for FDR-vs-threshold
    estimation
- name: HMMer
  role: Detects modification enzymes on ORF translations, defining which PTMs are
    applied to target and decoy precursor peptides
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:afbf48dd01fa5dfff0fd450c6d6ed3136f22f851be776e73642140faaf69f395
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ripp-target-decoy-database-construction

## Summary

Constructs target and decoy RiPP structure databases and uses decoy-vs-target peptide-spectrum match counts to estimate the false discovery rate of RiPP identifications, as implemented in MetaMiner (Cell Systems, 2019).

## When to use

Use when searching tandem mass spectra against predicted RiPP structures and a calibrated FDR (e.g., a 1% identification threshold) must be estimated via a target-decoy strategy.

## When NOT to use

- The spectra are not accompanied by paired genomics/metagenomics data (per the article, the other 99% of GNPS spectra cannot be searched).
- You need blind modification searches for cyclic peptides — MetaMiner does not perform these due to inherent computational complexity.
- Your precursor peptides have no sequence similarity to known RiPPs and you rely solely on the motif-ORF strategy, which misses such peptides (all-ORF mode is needed instead).

## Inputs

- Precursor peptide candidate sequences (translated ORFs from motif-ORF/all-ORF construction)
- HMMer profiles for RiPP modification enzymes
- Tandem mass spectra (e.g., GNPS datasets such as MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)
- Assembled contigs or metagenomic short reads (assembled into contigs by MetaMiner)

## Outputs

- Target RiPP structure database (predicted PTM-modified precursor peptides)
- Decoy RiPP structure database (shuffled precursor sequences with the same target modifications)
- Peptide-spectrum match p-values (computed with MS-DPR)
- FDR estimates (N_decoy/N_target) used to select the 1% FDR identification threshold

## How to apply

Start from the precursor peptide candidate sequences obtained via the motif-ORF/all-ORF construction (6-frame translated ORFs, default all-ORF length cutoff of 200 amino acids) for the selected genomic or metagenomic dataset. Run HMMer (Eddy, 2011) on the ORF translations to detect modification enzymes, and build the target RiPP database by applying the PTMs of each detected enzyme to its precursor peptides. Build the decoy database by randomly shuffling each peptide in the target precursor database BEFORE applying the same target modifications, so the decoys preserve the modification/mass profile while randomizing sequence. Search spectra against both databases with the modified Dereplicator engine, compute peptide-spectrum match p-values with MS-DPR, and estimate FDR = N_decoy/N_target by sweeping p-value thresholds; MetaMiner applies a 1% FDR threshold per dataset for RiPP identification.

## Related tools

- **MetaMiner** (Command-line tool / GNPS workflow that constructs the target and decoy RiPP precursor databases and computes FDR from the decoy database) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator** (Modified version used to search spectra against the database of putative target/decoy RiPPs)
- **MS-DPR** (Computes p-values of the peptide-spectrum matches used for FDR-vs-threshold estimation)
- **HMMer** (Detects modification enzymes on ORF translations, defining which PTMs are applied to target and decoy precursor peptides)

## Evaluation signals

- The decoy database must be built by shuffling target precursor sequences BEFORE applying the detected modification enzymes, so decoy and target peptides carry the same modification/mass profile.
- At the selected threshold, FDR must equal N_decoy/N_target computed from target and decoy peptide-spectrum matches.
- Positive-control check: on the isolated-RiPP dataset, all 18 known RiPPs should be correctly identified at the 1% FDR threshold.
- The 10 million GNPS spectra analyzed should yield 38 RiPP identifications at 1% FDR, clustered into 8071 spectral families.
- Identifications exceeding one modification per peptide should be penalized by the dynamic-programming scoring, restricting the number of modifications.

## Limitations

- The motif-ORF strategy is not sensitive: peptides without sequence similarity to known RiPPs are missed.
- No blind modification searches are performed for cyclic peptides due to inherent computational complexity.
- Genomes lacking sequence data (e.g., the lacticin producer) require searching spectra against previously described biosynthetic gene clusters rather than a genome.
- Only ~1% of GNPS spectra have paired genomics/metagenomics data, limiting the scope of the target-decoy search.

## Evidence
