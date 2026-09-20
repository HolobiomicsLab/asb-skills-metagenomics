---
name: peptide-sequence-shuffling-decoy-generation
description: Use when assigning tandem mass spectra (mzML/GNPS spectral datasets)
  to computationally predicted RiPP structures and you need an empirical FDR to set
  an acceptance threshold on peptide-spectrum matches.
when_to_use_negative:
- Spectra lack paired genomics/metagenomics data — the paper notes the other 99% of
  GNPS spectra cannot be searched, so no target/decoy peptidogenomic FDR is meaningful.
- Candidate compounds are non-ribosomal peptides or other non-RiPP classes — Dereplicator+
  family dereplication, not RiPP target-decoy peptidogenomics, is the appropriate
  route.
- Cyclic RiPPs requiring blind modification searches — the paper states blind modification
  searches are not currently performed for cyclic peptides due to computational complexity,
  so decoy-based scoring for such candidates is unsupported.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaMiner
  role: End-to-end pipeline that constructs the target precursor database, generates
    the shuffled decoy database, and computes FDR from target/decoy matches
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator
  role: Modified version used to search spectra against the database of putative target/decoy
    RiPPs
- name: MS-DPR
  role: Computes p-values of the peptide-spectrum matches used to sweep FDR thresholds
- name: HMMer
  role: Detects modification enzymes on translated ORFs, defining which PTMs are applied
    to both target and shuffled decoy precursors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:e4a7a1c56309d2123ca9607fd2ddde77aa9f19a5b9d56608c91cb5e878436922
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# peptide-sequence-shuffling-decoy-generation

## Summary

A target-decoy FDR estimation strategy for RiPP peptidogenomics: decoy RiPP precursor sequences are created by randomly shuffling each target precursor peptide BEFORE applying the same post-translational modifications, so decoys preserve the target modification/mass profile while randomizing the sequence. Comparing target vs decoy peptide-spectrum matches at a given threshold yields the FDR used to accept RiPP identifications.

## When to use

Use when assigning tandem mass spectra (mzML/GNPS spectral datasets) to computationally predicted RiPP structures and you need an empirical FDR to set an acceptance threshold on peptide-spectrum matches.

## When NOT to use

- Spectra lack paired genomics/metagenomics data — the paper notes the other 99% of GNPS spectra cannot be searched, so no target/decoy peptidogenomic FDR is meaningful.
- Candidate compounds are non-ribosomal peptides or other non-RiPP classes — Dereplicator+ family dereplication, not RiPP target-decoy peptidogenomics, is the appropriate route.
- Cyclic RiPPs requiring blind modification searches — the paper states blind modification searches are not currently performed for cyclic peptides due to computational complexity, so decoy-based scoring for such candidates is unsupported.

## Inputs

- Target RiPP precursor peptide sequences (translated ORFs from motif-ORF/all-ORF construction)
- Modification enzyme calls (detected with HMMer within 10 kbp putative biosynthetic gene cluster windows)
- Tandem mass spectra (e.g., GNPS spectral datasets such as MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)

## Outputs

- Decoy RiPP structure database (shuffled precursor sequences carrying the same PTM/mass profile as targets)
- Target vs decoy peptide-spectrum match p-values (computed with MS-DPR)
- FDR-vs-threshold table (FDR = N_decoy/N_target at each p-value threshold)
- Set of RiPP identifications passing the 1% FDR threshold

## How to apply

First build the target RiPP database: translate candidate ORFs (motif-ORF/all-ORF strategies), detect modification enzymes with HMMer, and apply the corresponding PTMs to each precursor peptide. Then, for every target precursor peptide, randomly shuffle its amino-acid sequence BEFORE applying the same target modifications — this is critical because it preserves the modification and mass profile while randomizing the sequence, making decoy scores comparable to target scores. Search the tandem mass spectra against both target and decoy databases using a modified version of Dereplicator; compute peptide-spectrum match p-values with MS-DPR. Sweep a grid of p-value thresholds and at each threshold compute FDR = N_decoy/N_target; in the MetaMiner paper, identifications are accepted at a 1% FDR threshold. Correct application means the decoy database has the same modification composition as the target database and the FDR-vs-threshold curve behaves smoothly so a 1% point can be reliably selected.

## Related tools

- **MetaMiner** (End-to-end pipeline that constructs the target precursor database, generates the shuffled decoy database, and computes FDR from target/decoy matches) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator** (Modified version used to search spectra against the database of putative target/decoy RiPPs)
- **MS-DPR** (Computes p-values of the peptide-spectrum matches used to sweep FDR thresholds)
- **HMMer** (Detects modification enzymes on translated ORFs, defining which PTMs are applied to both target and shuffled decoy precursors)

## Evaluation signals

- Decoy precursors carry the identical modification/mass profile as their target counterparts (shuffling happens before PTM application), so target and decoy score distributions are comparable.
- FDR is computed as N_decoy/N_target at each p-value threshold and the 1% FDR point is selectable; identifications are reported at this 1% threshold.
- Positive-control sanity check: MetaMiner recovered all 18 known RiPPs in the positive control dataset, so shuffled-decoy FDR should not suppress known true positives.
- Every reported RiPP identification can be traced to a peptide-spectrum match with an MS-DPR p-value below the threshold chosen from the FDR-vs-threshold curve.
- Decoy database size and PTM composition mirror the target database (one shuffled counterpart per target precursor).

## Limitations

- Motif-ORF precursor discovery is not sensitive; peptides without sequence similarity to known RiPPs are missed, shrinking the target set and affecting FDR estimates.
- Blind modification searches are not performed for cyclic peptides due to computational complexity, so decoy-based FDR is unavailable for those cases.
- Genome sequence may be unavailable for some producers (e.g., the lacticin producer), forcing searches against previously described biosynthetic gene clusters rather than a full genome-derived target/decoy pair.
- Only 38 RiPPs were identified at 1% FDR from 10 million spectra, because most GNPS spectra lack paired genomics/metagenomics data and many spectral families belong to non-RiPP classes.

## Evidence

- [readme] Tool availability via GNPS workflow: [evidence span withheld — anchor status: not_found]
