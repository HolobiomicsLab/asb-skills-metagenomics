---
name: gnps-dataset-acquisition-and-curation
description: Use when reproducing or extending the MetaMiner peptidogenomics pipeline
  and you need to identify which GNPS MSVxxxxxx accessions have paired genomic/metagenomic
  data (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552,
  MSV000080102) and stage them for RiPP search.
when_to_use_negative:
- The GNPS dataset lacks paired genomic/metagenomic data — 99% of GNPS spectra are
  not accompanied by genomics/metagenomics data, making peptidogenomic search impossible
- The goal is dereplication of non-ribosomal peptides rather than RiPP discovery —
  those spectral families are better searched against known structures with Dereplicator+
- You already have assembled genomes or known biosynthetic gene clusters and need
  no dataset pairing/curation step — go directly to precursor database construction
  and spectral search
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMiner
  role: Command-line tool / GNPS RIPPQUEST workflow that searches mass spectra against
    genomic and metagenomic data for RiPP discovery
  repo: https://github.com/mohimanilab/MetaMiner
- name: GNPS
  role: Molecular networking infrastructure hosting the mass-spectrometry datasets
    to be acquired and the hosted MetaMiner workflow
- name: Dereplicator
  role: Modified version used to score peptide-spectrum matches against target/decoy
    RiPP databases
- name: HMMer
  role: Detects modification enzymes on translated sequences to delimit putative biosynthetic
    gene clusters
- name: MS-DPR
  role: Computes p-values of peptide-spectrum matches for FDR estimation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:40223b7fd2a1436d7d65e690c8f0df27186b93edb7a484c4a78b8a382c2c5ba7
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gnps-dataset-acquisition-and-curation

## Summary

Acquire the GNPS mass-spectrometry datasets that are explicitly paired with genomic or metagenomic sequencing data, and curate them into the per-dataset inputs required to reproduce MetaMiner's RiPP identification table (38 RiPPs at 1% FDR from 10 million spectra). Only spectra with paired omics data can be peptidogenomically searched, so dataset selection is the gating step.

## When to use

Use when reproducing or extending the MetaMiner peptidogenomics pipeline and you need to identify which GNPS MSVxxxxxx accessions have paired genomic/metagenomic data (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102) and stage them for RiPP search.

## When NOT to use

- The GNPS dataset lacks paired genomic/metagenomic data — 99% of GNPS spectra are not accompanied by genomics/metagenomics data, making peptidogenomic search impossible
- The goal is dereplication of non-ribosomal peptides rather than RiPP discovery — those spectral families are better searched against known structures with Dereplicator+
- You already have assembled genomes or known biosynthetic gene clusters and need no dataset pairing/curation step — go directly to precursor database construction and spectral search

## Inputs

- GNPS molecular networking dataset accessions with paired genomic/metagenomic data (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102)
- Paired genomic or metagenomic short-read sequencing data for each GNPS dataset
- MetaMiner command-line tool or the GNPS-hosted RIPPQUEST workflow

## Outputs

- Per-dataset paired spectra/genomics input bundles staged for MetaMiner peptidogenomic search
- Curated RiPP identification table at 1% FDR (38 RiPPs: 31 known, 7 unknown, from 10 million spectra)
- Per-dataset FDR estimates from the target-decoy approach

## How to apply

First, obtain the MetaMiner command-line tool from github.com/mohimanilab/MetaMiner (or run the RIPPQUEST workflow hosted on GNPS), then download the paired GNPS spectral datasets — the article analyzes spectra from eight genomic/metagenomic datasets including MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552 (Nguyen et al., 2013 cyanobacterial data), and MSV000080102 (21 International Space Station isolates). For each dataset, pair the mass spectra with its corresponding short-read genomic or metagenomic input, since MetaMiner assembles short reads into contigs before 6-frame translation and precursor-peptide construction via the motif-ORF and all-ORF strategies (flagging ORFs below the default 200-amino-acid threshold in all-ORF mode). Search spectra against target and decoy RiPP databases using the modified Dereplicator scoring, penalizing matches with more than one modification, compute peptide-spectrum match p-values with MS-DPR, and estimate FDR per dataset with the target-decoy approach at the 1% threshold. Enlarge the identification set via spectral networks, then curate surviving identifications into a single table and verify it matches the reported composition of 38 RiPPs (31 known, 7 unknown). The special case of the lacticin producer — whose genome is unavailable — should be searched against its previously described biosynthetic gene clusters rather than a genome.

## Related tools

- **MetaMiner** (Command-line tool / GNPS RIPPQUEST workflow that searches mass spectra against genomic and metagenomic data for RiPP discovery) — https://github.com/mohimanilab/MetaMiner
- **GNPS** (Molecular networking infrastructure hosting the mass-spectrometry datasets to be acquired and the hosted MetaMiner workflow)
- **Dereplicator** (Modified version used to score peptide-spectrum matches against target/decoy RiPP databases)
- **HMMer** (Detects modification enzymes on translated sequences to delimit putative biosynthetic gene clusters)
- **MS-DPR** (Computes p-values of peptide-spectrum matches for FDR estimation)

## Examples

```
git clone https://github.com/mohimanilab/MetaMiner  # then stage paired GNPS datasets MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102
```

## Evaluation signals

- All acquired datasets are GNPS accessions with explicit paired genomic/metagenomic data (e.g. MSV000079506 with MSV000079622; MSV000078839 with MSV000078604; MSV000078552; MSV000080102 ISS isolates)
- Curated identification table contains exactly 38 RiPPs (31 known including AIP, Mec-PSM, delta-toxin, polytheonamide, wewakezole, lichenicidin, plus 7 unknown) at the 1% FDR threshold
- Positive control check: MetaMiner correctly identifies all 18 known RiPPs in the isolated-RiPP control dataset
- Every reported identification is supported by a target-decoy FDR estimate at or below 1% computed per dataset via MS-DPR p-values
- Identification set is enlarged consistently via spectral networks, with the 10 million spectra clustering into 8071 spectral families as a sanity anchor

## Limitations

- 99% of GNPS spectra are not accompanied by genomics/metagenomics data and cannot be searched peptidogenomically
- No blind modification searches are performed for cyclic peptides due to computational complexity
- The motif-ORF strategy is not sensitive and misses precursor peptides without sequence similarity to known RiPPs
- When the producer genome is unavailable (e.g. lacticin), the spectrum must be searched against previously described biosynthetic gene clusters instead

## Evidence

- [readme] Tool availability and hosted workflow: [evidence span withheld — anchor status: not_found]
