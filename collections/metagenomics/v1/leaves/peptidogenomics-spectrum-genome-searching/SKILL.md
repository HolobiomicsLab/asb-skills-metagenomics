---
name: peptidogenomics-spectrum-genome-searching
description: Use when you have tandem MS spectra (e.g. GNPS datasets) paired with
  producer genomes, RefSeq assemblies, or metagenomic/metatranscriptomic short reads,
  and want to identify RiPPs (lanthipeptides, lassopeptides, LAPs, linaridins, glycocins,
  cyanobactins, proteusins) at 1% FDR.
when_to_use_negative:
- Spectra are not accompanied by paired genomics/metagenomics data (about 99% of GNPS
  spectra), making spectrum-to-genome searching impossible.
- The goal is blind modification search for cyclic peptides — MetaMiner does not perform
  blind modification searches for cyclic peptides due to inherent computational complexity.
- The spectral families are non-ribosomal peptides — Dereplicator+ analysis shows
  many GNPS spectral families belong to non-ribosomal peptides, outside the RiPP scope
  of this skill.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMiner
  role: 'Core peptidogenomics pipeline: 6-frame translation, HMMer modification-enzyme
    detection, precursor database construction (motif-ORF/all-ORF), decoy generation,
    and spectrum search'
  repo: https://github.com/mohimanilab/MetaMiner
- name: Dereplicator
  role: Modified dynamic-programming search engine used by MetaMiner to score spectra
    against target/decoy RiPP precursor databases with restricted blind modifications
- name: HMMer
  role: Detects RiPP modification enzymes in translated genome/metagenome sequences
    to define putative biosynthetic gene cluster windows
- name: MS-DPR
  role: Computes p-values of the peptide-spectrum matches used for the target-decoy
    FDR estimate
- name: GNPS
  role: Hosts the MetaMiner/RIPPQUEST workflow and the molecular networking spectral
    datasets searched against genomic data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:16a4418aa9757c705fb9fb88fe4590246e25ea72856b96300fb06d0223e1a6e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - Dereplicator
  - GNPS
  - HMMer
  - MS-DPR
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# peptidogenomics-spectrum-genome-searching

## Summary

A peptidogenomics skill for searching tandem MS spectra (e.g. from GNPS molecular networking) against putative RiPP precursor peptides derived from genomic or metagenomic sequence, enabling discovery of ribosomally synthesized and post-translationally modified peptides with blind modifications. It integrates computational mass spectrometry with genome mining to link spectra to biosynthetic gene clusters.

## When to use

Use when you have tandem MS spectra (e.g. GNPS datasets) paired with producer genomes, RefSeq assemblies, or metagenomic/metatranscriptomic short reads, and want to identify RiPPs (lanthipeptides, lassopeptides, LAPs, linaridins, glycocins, cyanobactins, proteusins) at 1% FDR.

## When NOT to use

- Spectra are not accompanied by paired genomics/metagenomics data (about 99% of GNPS spectra), making spectrum-to-genome searching impossible.
- The goal is blind modification search for cyclic peptides — MetaMiner does not perform blind modification searches for cyclic peptides due to inherent computational complexity.
- The spectral families are non-ribosomal peptides — Dereplicator+ analysis shows many GNPS spectral families belong to non-ribosomal peptides, outside the RiPP scope of this skill.

## Inputs

- Tandem mass spectrometry (MS/MS) spectra, e.g. GNPS datasets such as MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102
- Producer genomes (e.g. NCBI RefSeq) or metagenomic/metatranscriptomic short reads from microbiome samples
- Previously described biosynthetic gene clusters (used when the producer genome is unavailable, e.g. lacticin)

## Outputs

- RiPP identifications (peptide-spectrum matches) accepted at 1% FDR, with p-values from MS-DPR
- Positive-control identification lists (e.g. positive_control_results.csv with hit/miss per spectrum)
- Spectral-network-enlarged families of RiPP identifications and their analogs

## How to apply

Run the MetaMiner pipeline (github.com/mohimanilab/MetaMiner, also deployed as the RIPPQUEST workflow on GNPS): 6-frame-translate the DNA (assembling short reads into contigs first if the input is raw reads), then run HMMer to detect RiPP modification enzymes and build a 10 kbp window (putative biosynthetic gene cluster) centered on each enzyme. Build a precursor peptide database using both the motif-ORF and all-ORF strategies (all-ORF flags ORFs shorter than a pre-defined threshold, default 200 amino acids), and create a decoy database by randomly shuffling each target precursor peptide. Score peptide-spectrum matches with MetaMiner's modified Dereplicator dynamic-programming search, which restricts the number of blind modifications and penalizes matches with more than one modification, then compute p-values with MS-DPR and FDR with the target-decoy approach. Accept identifications at a 1% FDR threshold, and enlarge the set of RiPP identifications via spectral networking.

## Related tools

- **MetaMiner** (Core peptidogenomics pipeline: 6-frame translation, HMMer modification-enzyme detection, precursor database construction (motif-ORF/all-ORF), decoy generation, and spectrum search) — https://github.com/mohimanilab/MetaMiner
- **Dereplicator** (Modified dynamic-programming search engine used by MetaMiner to score spectra against target/decoy RiPP precursor databases with restricted blind modifications)
- **HMMer** (Detects RiPP modification enzymes in translated genome/metagenome sequences to define putative biosynthetic gene cluster windows)
- **MS-DPR** (Computes p-values of the peptide-spectrum matches used for the target-decoy FDR estimate)
- **GNPS** (Hosts the MetaMiner/RIPPQUEST workflow and the molecular networking spectral datasets searched against genomic data)

## Examples

```
Run MetaMiner as the RIPPQUEST workflow on GNPS: https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22%3A%22RIPPQUEST%22%7D (clone source: git clone https://github.com/mohimanilab/MetaMiner)
```

## Evaluation signals

- Positive-control benchmark: MetaMiner correctly identifies all 18 of 18 known RiPPs (prochlorosins, geobacillin, sublancin, haloduracin, lacticin, bicereusins, flavecins) at the 1% FDR threshold.
- FDR estimate via target-decoy approach stays at or below the 1% threshold for accepted identifications.
- Precursor database follows both motif-ORF and all-ORF strategies, with all-ORF ORFs shorter than the 200 amino-acid default threshold and decoys generated by random shuffling of each target peptide.
- Identified spectra match the expected modified RiPP structure (correct precursor mass plus one or few blind modifications, with matches carrying more than one modification penalized).
- Identified RiPPs can be enlarged into spectral-network families with matching analogs (e.g. seven novel RiPPs with numerous analogs from eight spectral datasets).

## Limitations

- About 99% of spectra in GNPS are not accompanied by genomics/metagenomics data, making spectrum-to-genome search impossible for them.
- No blind modification searches are performed for cyclic peptides due to inherent computational complexity.
- The motif-ORF strategy is not sensitive and misses peptides without sequence similarity to known RiPPs.
- When the producer genome is unavailable (e.g. lacticin), the spectrum must be searched against previously described biosynthetic gene clusters rather than a genome.

## Evidence
