---
name: population-genomics-divergence-counting
description: 'Use when the goal is to produce per-gene polymorphism (DAF spectrum:
  Pi, P0) and divergence (mi, Di, m0, D0) count tables from FASTA alignments or VCF
  population data as input to MKT-based estimation of adaptive evolution (alpha).'
when_to_use_negative:
- Input is already a formatted iMKT 'daf'/'divergence' table (e.g. the myDafData/myDivergenceData
  sample data frames) — go directly to the MK test instead of re-extracting counts.
- Only DAF polymorphism data are available without divergence counts — the MKT framework
  requires paired polymorphism and divergence tables.
- Data are non-protein-coding (e.g. non-coding genomic regions), since the 0-fold/4-fold
  functional-class decomposition that defines the selected vs. neutral classes does
  not apply.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: iMKTData
  role: Python/bash/Jupyter pipelines that extract the unfolded site frequency spectrum
    from 1000GP VCF and DGN alignments and produce the tab-delimited count tables
    per gene and functional class
  repo: https://github.com/BGD-UAB/iMKTData
- name: iMKT (R package)
  role: Downstream consumer of the daf and divergence data frames; computes MK-derived
    estimates of alpha and the fractions of deleterious/neutral sites
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT web service
  role: Web-based service that runs the four MKT types on the generated count tables
    or pre-loaded D. melanogaster/human gene sets
- name: PopFly
  role: Source of D. melanogaster variation (DGN) and D. melanogaster–D. simulans
    divergence data in FASTA format
- name: PopHuman
  role: Source of human 1000GP Phase III variation and human–chimpanzee divergence
    data in VCF format
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:6d39ab7cafed8378b7ee381041745180b401fab95dd0ca5c0dc8527217ba3c40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGD-UAB/iMKTData
  tools:
  - PopFly
  - PopHuman
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  - iMKT web service
  - iMKTData
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# population-genomics-divergence-counting

## Summary

A method for extracting unfolded site frequency spectra and per-gene divergence counts from population genomic alignments (DGN/PopFly FASTA, 1000GP/PopHuman VCF), classifying sites into MKT functional classes (0-fold: selected; 4-fold: neutral) to feed McDonald-Kreitman analyses such as those implemented in iMKT.

## When to use

Use when the goal is to produce per-gene polymorphism (DAF spectrum: Pi, P0) and divergence (mi, Di, m0, D0) count tables from FASTA alignments or VCF population data as input to MKT-based estimation of adaptive evolution (alpha).

## When NOT to use

- Input is already a formatted iMKT 'daf'/'divergence' table (e.g. the myDafData/myDivergenceData sample data frames) — go directly to the MK test instead of re-extracting counts.
- Only DAF polymorphism data are available without divergence counts — the MKT framework requires paired polymorphism and divergence tables.
- Data are non-protein-coding (e.g. non-coding genomic regions), since the 0-fold/4-fold functional-class decomposition that defines the selected vs. neutral classes does not apply.

## Inputs

- D. melanogaster FASTA alignments from DGN/PopFly (variation) plus D. melanogaster–D. simulans divergence data
- Human 1000GP Phase III VCF with variant ancestral-state annotations
- Divergence data between humans and chimpanzees from PopHuman (VCF)
- Recombination maps (Comeron et al. 2012 for Drosophila; Bhérer et al. 2017 for human)
- 1000GP accessible genome pilot mask for low-quality variant exclusion

## Outputs

- Tab-delimited file with unfolded site frequency spectrum (DAF, Pi, P0) per gene/population
- Tab-delimited divergence counts (mi, Di, m0, D0) per gene, with analyzable sites by largest transcript
- Counts stratified by MKT functional classes (0-fold: selected class; 4-fold: neutral class), ready for iMKT web service or R package

## How to apply

Run the iMKTData pipelines to convert raw population genomic data into analysis-ready counts: for D. melanogaster, retrieve variation data from the Drosophila Genome Nexus via PopFly in FASTA format plus divergence between D. melanogaster and D. simulans, and recombination data from Comeron et al. 2012; for human, retrieve 1000GP Phase III VCF with ancestral-state information and human-chimpanzee divergence from PopHuman, recombination data from Bhérer et al. 2017, and apply the 1000GP pilot mask to exclude low-quality variants. Classify each site into 0-fold degenerate (selected) and 4-fold degenerate (neutral) functional classes and compute the unfolded DAF spectrum (Pi, P0 per frequency bin), analyzed sites (mi, m0 by largest transcript), and divergence counts (Di, D0) per gene, population, and functional class. Output a tab-delimited file consumable by the iMKT web service or R package, where functions such as standardMKT expect a 'daf' data frame (daf, Pi, P0) and a 'divergence' data frame (mi, Di, m0, D0). The step-by-step Python 3.6/Jupyter notebooks (dmelProteins.ipynb, humanProteins.ipynb) document each pipeline stage and can be parallelized with custom bash scripts.

## Related tools

- **iMKTData** (Python/bash/Jupyter pipelines that extract the unfolded site frequency spectrum from 1000GP VCF and DGN alignments and produce the tab-delimited count tables per gene and functional class) — https://github.com/BGD-UAB/iMKTData
- **iMKT (R package)** (Downstream consumer of the daf and divergence data frames; computes MK-derived estimates of alpha and the fractions of deleterious/neutral sites) — https://github.com/BGD-UAB/iMKT
- **iMKT web service** (Web-based service that runs the four MKT types on the generated count tables or pre-loaded D. melanogaster/human gene sets)
- **PopFly** (Source of D. melanogaster variation (DGN) and D. melanogaster–D. simulans divergence data in FASTA format)
- **PopHuman** (Source of human 1000GP Phase III variation and human–chimpanzee divergence data in VCF format)

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Output file is a well-formed tab-delimited table containing the unfolded DAF spectrum and per-gene divergence counts (Dn/Ds/Pn/Ps equivalents: Di, D0, Pi, P0).
- Counts are stratified into the two MKT functional classes: 0-fold degenerate (selected) and 4-fold degenerate (neutral).
- Analyzable sites are reported per largest transcript and stratified by population, matching the pipeline specification.
- Loading the output as the iMKT R package's 'daf' (daf, Pi, P0) and 'divergence' (mi, Di, m0, D0) data frames succeeds and standardMKT returns alpha with a Fisher's exact test P-value and MKT table.
- Human pipeline variants are filtered with the 1000GP accessible genome pilot mask, so masked low-quality positions contribute zero counts.

## Limitations

- Pipeline paths must be edited manually for local execution, and multiple steps require user-written bash scripts to parallelize; the reference environment was a 100GB RAM, 16-core Intel Xeon server with a conda environment (imktData.yml).
- The iMKTData repository contains only raw code to reproduce main results — notebooks and src folders must be downloaded/overwritten and cannot be run as-is without the downloaded external data files.
- The unfolded SFS requires ancestral-state information (e.g. derived allele polarization for Drosophila via D. simulans and for human via 1000GP annotations); mis-specification of ancestral state would fold the spectrum and invalidate the MKT counts.

## Evidence

- [readme] Pipeline returns tab-delimited counts: "Pipelines to extract unfolded site frequency spectrum from 1000GP VCF and DGN alignments."
- [readme] Output format of iMKTData pipelines: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional"
- [methods] Drosophila data sources: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [methods] Human filter step: "[Pilot mask](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/supporting/accessible_genome_masks) to exclude low quality variants download from 1000GP ftp"
- [readme] iMKT function signature for count tables: [evidence span withheld — anchor status: not_found]
