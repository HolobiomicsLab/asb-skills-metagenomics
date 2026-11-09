---
name: post-translational-modification-application-to-precursors
description: Use when building a target/decoy RiPP database from precursor peptide
  ORFs whose biosynthetic gene cluster modification enzymes (lanthipeptide, lassopeptide,
  LAP, linaridin, glycocin, cyanobactin, proteusin) have already been identified.
when_to_use_negative:
- The spectra lack paired genomics/metagenomics data — per the article, 99% of GNPS
  spectra cannot be searched this way.
- Candidate RiPPs are cyclic peptides requiring blind modification searches, which
  MetaMiner does not currently support due to computational complexity.
- Precursor peptides have no sequence similarity to known RiPPs and the motif-ORF
  strategy would miss them; use all-ORF mode instead.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: HMMer
  role: Detects the modification enzymes in translated ORFs, which determines which
    PTMs are applied to each precursor peptide
- name: Dereplicator
  role: MetaMiner uses a modified version of Dereplicator to search spectra against
    the database of putative target/decoy RiPPs built by PTM application
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches used for FDR threshold sweeping
- name: MetaMiner
  role: Orchestrates target/decoy RiPP database construction and searching; available
    as a command line tool and GNPS RIPPQUEST workflow
  repo: https://github.com/mohimanilab/MetaMiner
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:8ad76768fe437d6e0c04fd235bd7fad6f7582b7f47aafb0bf6e3ce76fb8dc91f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# post-translational-modification-application-to-precursors

## Summary

This skill constructs the searchable RiPP structure database by applying the post-translational modifications of each HMMer-detected modification enzyme to its precursor peptide(s), and analogously to shuffled decoy precursors for FDR estimation.

## When to use

Use when building a target/decoy RiPP database from precursor peptide ORFs whose biosynthetic gene cluster modification enzymes (lanthipeptide, lassopeptide, LAP, linaridin, glycocin, cyanobactin, proteusin) have already been identified.

## When NOT to use

- The spectra lack paired genomics/metagenomics data — per the article, 99% of GNPS spectra cannot be searched this way.
- Candidate RiPPs are cyclic peptides requiring blind modification searches, which MetaMiner does not currently support due to computational complexity.
- Precursor peptides have no sequence similarity to known RiPPs and the motif-ORF strategy would miss them; use all-ORF mode instead.

## Inputs

- Precursor peptide candidate sequences (translated ORFs from motif-ORF/all-ORF construction)
- List of HMMer-detected modification enzymes per putative biosynthetic gene cluster (10 kbp window)
- Tandem mass spectra (GNPS datasets such as MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)

## Outputs

- Target RiPP structure database (modified precursor peptide sequences with PTM/mass profiles)
- Decoy RiPP structure database (shuffled precursors with identical PTM profiles)
- Peptide-spectrum match p-values and FDR-vs-threshold estimates (e.g. fdr_vs_threshold.csv) used for 1% FDR filtering

## How to apply

First run HMMer (Eddy, 2011) against the 6-frame-translated ORFs (from motif-ORF or all-ORF construction, default ORF length < 200 aa) to detect all modification enzymes in each putative 10 kbp biosynthetic gene cluster window. Build the target RiPP database by applying the PTMs corresponding to each detected modification enzyme to its precursor peptide(s), so each candidate structure carries the correct modification/mass profile (e.g. dehydrations, thioether cross-links, macrocyclations) expected for that enzyme class. Build the decoy database by randomly shuffling each target precursor peptide sequence BEFORE applying the same target modifications, preserving the modification/mass profile while randomizing the amino-acid order. Search tandem mass spectra against both target and decoy databases using a modified version of Dereplicator, with peptide-spectrum match p-values computed via MS-DPR, then sweep p-value thresholds counting decoy vs target matches to estimate FDR = Ndecoy/Ntarget and apply the 1% FDR threshold.

## Related tools

- **HMMer** (Detects the modification enzymes in translated ORFs, which determines which PTMs are applied to each precursor peptide)
- **Dereplicator** (MetaMiner uses a modified version of Dereplicator to search spectra against the database of putative target/decoy RiPPs built by PTM application)
- **MS-DPR** (Computes p-values of peptide-spectrum matches used for FDR threshold sweeping)
- **MetaMiner** (Orchestrates target/decoy RiPP database construction and searching; available as a command line tool and GNPS RIPPQUEST workflow) — https://github.com/mohimanilab/MetaMiner

## Evaluation signals

- Decoy precursor sequences are shuffled versions of target precursors but carry the identical modification/mass profile as their targets.
- At the 1% FDR threshold, FDR = Ndecoy/Ntarget should hold for the chosen p-value cutoff in the FDR-vs-threshold table.
- Positive control check: on a dataset of isolated RiPPs, the modified structures should recover all 18 known RiPPs (MetaMiner's reported benchmark).
- Every searched spectrum match maps to a precursor whose PTMs correspond to a modification enzyme detected by HMMer in its 10 kbp BGC window.
- Matches with more than one modification are penalized by the dynamic-programming scoring, so high-scoring multi-modification hits should be down-weighted.

## Limitations

- No blind modification searches are performed for cyclic peptides due to inherent computational complexity.
- The motif-ORF strategy is not sensitive and misses precursor peptides without sequence similarity to known RiPPs.
- If the producer's genome is unavailable (e.g. the lacticin producer), PTM application must rely on previously described biosynthetic gene clusters rather than detected enzymes.

## Evidence
