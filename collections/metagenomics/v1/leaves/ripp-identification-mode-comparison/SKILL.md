---
name: ripp-identification-mode-comparison
description: Decision support for choosing a MetaMiner database-construction mode
  when RiPPs must be identified from spectra with paired genomic data and the analyst
  must weigh sensitivity for novel RiPPs (motif-ORF) against broader ORF enumeration
  (all-ORF) at strict FDR.
when_to_use_negative:
- The spectra lack paired genomics/metagenomics data — per the article, ~99% of GNPS
  spectra cannot be searched against genomes, making peptidogenomic mode comparison
  inapplicable.
- The goal is discovery of non-ribosomal peptides — Dereplicator+ analysis showed
  many spectral families belong to non-ribosomal peptide classes outside MetaMiner's
  RiPP scope.
- The candidate RiPPs have no sequence similarity to known RiPPs and the analysis
  depends solely on motif-ORF mode, which the article states misses such peptides.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0154
tools:
- name: MetaMiner
  role: 'Core peptidogenomics tool: builds motif-ORF/all-ORF precursor databases,
    searches spectra against target/decoy RiPP databases with modified Dereplicator
    scoring'
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Detects modification enzymes on 6-frame-translated sequences to define the
    10-kbp putative BGC windows used in motif-ORF mode
- name: Dereplicator
  role: Modified version used by MetaMiner for scoring peptide-spectrum matches against
    the putative target/decoy RiPP database
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches used for target-decoy FDR estimation
- name: antiSMASH
  role: Genome mining of biosynthetic gene clusters (e.g., identified 70 BGCs including
    29 RiPP-encoding clusters in the analyzed genomes)
- name: GNPS
  role: Molecular networking infrastructure hosting the searched spectra and the MetaMiner
    workflow; enables spectral networking enlargement of identifications
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:f4a5f18e16b06281337fb3d5b0b5b409ce095d3b71f9d45d44c02d425cf3a286
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
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ripp-identification-mode-comparison

## Summary

Compares MetaMiner's motif-ORF versus all-ORF precursor-peptide database construction modes for RiPP identification from spectra paired with genomes/metagenomes, evaluated at target-decoy FDR thresholds of 0% and 1%. It quantifies how genome-mining mode sensitivity trades off against known/unknown RiPP discovery yield.

## When to use

Decision support for choosing a MetaMiner database-construction mode when RiPPs must be identified from spectra with paired genomic data and the analyst must weigh sensitivity for novel RiPPs (motif-ORF) against broader ORF enumeration (all-ORF) at strict FDR.

## When NOT to use

- The spectra lack paired genomics/metagenomics data — per the article, ~99% of GNPS spectra cannot be searched against genomes, making peptidogenomic mode comparison inapplicable.
- The goal is discovery of non-ribosomal peptides — Dereplicator+ analysis showed many spectral families belong to non-ribosomal peptide classes outside MetaMiner's RiPP scope.
- The candidate RiPPs have no sequence similarity to known RiPPs and the analysis depends solely on motif-ORF mode, which the article states misses such peptides.

## Inputs

- GNPS spectral datasets MSV000078839 and MSV000078604 (ACTI Actinomycetes tandem mass spectra)
- 36 paired RefSeq genomes
- Simulated metagenome short reads
- Precursor peptide target and shuffled decoy databases in motif-ORF and all-ORF modes

## Outputs

- Counts of identified known and unknown RiPPs per mode (motif-ORF, all-ORF) × FDR threshold (0%, 1%)
- Comparison table written to acti_ripp_mode_fdr_comparison.csv
- FDR-estimated peptide-spectrum matches with MS-DPR p-values

## How to apply

Retrieve the spectral dataset (e.g., ACTI Actinomycetes GNPS datasets MSV000078839 and MSV000078604) with paired RefSeq genomes or simulated metagenome short reads, assemble short reads into contigs, and 6-frame translate. Clone github.com/mohimanilab/MetaMiner and build two precursor peptide databases: in motif-ORF mode, run HMMer to detect modification enzymes and define 10-kbp putative biosynthetic gene cluster windows centered at each enzyme; in all-ORF mode, enumerate ORFs shorter than the default 200 amino acid threshold. Search the spectra against each target database and its decoy (generated by randomly shuffling each precursor peptide) using MetaMiner's modified Dereplicator scoring, compute peptide-spectrum match p-values with MS-DPR, and estimate FDR with the target-decoy approach at 0% and 1% thresholds. Enlarge identifications via spectral networking, then count known and unknown RiPPs per mode × FDR combination. The rationale is that motif-ORF constrains candidates to RiPP-similar motifs near modification enzymes while all-ORF casts a wider net; the mode choice measurably changes which RiPPs pass strict FDR filtering.

## Related tools

- **MetaMiner** (Core peptidogenomics tool: builds motif-ORF/all-ORF precursor databases, searches spectra against target/decoy RiPP databases with modified Dereplicator scoring) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Detects modification enzymes on 6-frame-translated sequences to define the 10-kbp putative BGC windows used in motif-ORF mode)
- **Dereplicator** (Modified version used by MetaMiner for scoring peptide-spectrum matches against the putative target/decoy RiPP database)
- **MS-DPR** (Computes p-values of peptide-spectrum matches used for target-decoy FDR estimation)
- **antiSMASH** (Genome mining of biosynthetic gene clusters (e.g., identified 70 BGCs including 29 RiPP-encoding clusters in the analyzed genomes))
- **GNPS** (Molecular networking infrastructure hosting the searched spectra and the MetaMiner workflow; enables spectral networking enlargement of identifications)

## Examples

```
git clone https://github.com/mohimanilab/MetaMiner && python metaminer.py --mode motif-ORF --spectra MSV000078839 --fdr 0.01
```

## Evaluation signals

- At 0% FDR on the ACTI dataset, motif-ORF mode identifies 3 unknown plus 5 known RiPPs while all-ORF mode identifies only 2 known RiPPs.
- At 1% FDR on the ACTI dataset, 8 RiPPs total are identified in this dataset.
- The comparison table (acti_ripp_mode_fdr_comparison.csv) contains one row per mode × FDR threshold combination with known/unknown RiPP counts.
- Decoy database is built by random shuffling of every peptide in the target precursor database, and reported FDR is derived from target-decoy counting, not a fixed p-value cutoff.
- Positive-control sanity: in the article's isolated-RiPP control dataset, the pipeline correctly identified all 18 known RiPPs.

## Limitations

- The motif-ORF strategy is not sensitive and misses precursor peptides without sequence similarity to known RiPPs.
- No blind modification searches are performed for cyclic peptides due to computational complexity.
- Only 38 RiPPs were identified after searching 10 million spectra against eight genomic/metagenomic datasets; ~99% of GNPS spectra lack paired genomics/metagenomics data and cannot be searched.
- All-ORF mode is restricted to ORFs shorter than the default 200 amino acid threshold, so longer precursor ORFs are excluded.
- Matches with more than one modification are penalized and the number of modifications is restricted by the dynamic-programming scoring approach.

## Evidence
