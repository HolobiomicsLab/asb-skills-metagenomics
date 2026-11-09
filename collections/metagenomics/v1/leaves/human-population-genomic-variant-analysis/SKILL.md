---
name: human-population-genomic-variant-analysis
description: Use when you have human population VCF-derived unfolded SFS / MKT count
  tables (e.g. 1000GP populations like CEU or YRI) and need to estimate α or compare
  selection regimes between a candidate gene set and the genome-wide distribution.
when_to_use_negative:
- Input data are Drosophila (DGN/PopFly FASTA) alignments rather than human 1000GP
  VCF-derived tables — use the D. melanogaster pipeline with Comeron et al. 2012 recombination
  data instead
- You only have raw VCFs without ancestral-state information or functional-class annotation
  (0-fold/4-fold), since MKT tests require unfolded SFS and divergence counts
- The question concerns weakly/strongly deleterious fractions without the polymorphism-divergence
  structure (daf + divergence tables) required by iMKT functions
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
tools:
- name: iMKT
  role: Web-based service performing the four MKT types (standard MKT, FWWMKT, eMKT,
    aMKT) on pre-loaded human (PopHuman) and Drosophila (PopFly) data or user data
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: R functions (e.g. standardMKT, eMKT) that take daf and divergence data frames
    and return α plus Fisher's exact test P-value and MKT/omega tables
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Used in the 02_Human pipeline Jupyter notebook to extract unfolded SFS and
    divergence counts from 1000GP VCFs
  repo: https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- name: bash
  role: Raw scripts in src/ of the iMKTData repo used to execute/parallelize the human
    pipeline steps
  repo: https://github.com/BGD-UAB/iMKTData
- name: Jupyter
  role: Notebook (02_Human pipeline, Python 3.6 kernel) running the human pipeline
    step by step
  repo: https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:c5cf01ce00e4a072b89eda58858080e0798c12941cb238176e4c46540564a4d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# human-population-genomic-variant-analysis

## Summary

Analyzes human population genomic variation (1000GP Phase III VCF from PopHuman) together with human–chimpanzee divergence to compute McDonald–Kreitman-derived estimates of adaptation (α) and other selection regimes for user-defined gene sets, e.g. high-recombination vs whole-genome comparisons using Bhèrer et al. 2017 recombination maps.

## When to use

Use when you have human population VCF-derived unfolded SFS / MKT count tables (e.g. 1000GP populations like CEU or YRI) and need to estimate α or compare selection regimes between a candidate gene set and the genome-wide distribution.

## When NOT to use

- Input data are Drosophila (DGN/PopFly FASTA) alignments rather than human 1000GP VCF-derived tables — use the D. melanogaster pipeline with Comeron et al. 2012 recombination data instead
- You only have raw VCFs without ancestral-state information or functional-class annotation (0-fold/4-fold), since MKT tests require unfolded SFS and divergence counts
- The question concerns weakly/strongly deleterious fractions without the polymorphism-divergence structure (daf + divergence tables) required by iMKT functions

## Inputs

- 1000GP Phase III VCF files with ancestral-state annotations retrieved from PopHuman
- 1000GP pilot mask (accessible genome masks) from the 1000GP FTP to exclude low-quality variants
- Bhèrer et al. 2017 human recombination rate data for gene-set stratification
- Population definition (e.g. CEU, YRI) from 1000GP Phase III
- Tab-deliterated MKT input tables: daf data frame (daf, Pi, P0) and divergence data frame (mi, Di, m0, D0) with 0-fold and 4-fold functional classes

## Outputs

- Tab-deliterated per-gene unfolded SFS, analyzable sites (by largest transcript), and divergence counts by gene, population, and MKT functional class
- Alpha (α) estimates per population and gene set (e.g. emkt_high_recombination_ceu_yri_results.tsv)
- MKT table (polymorphism/divergence for neutral and selected classes), Ka, Ks, omega, omegaA, omegaD, and Fisher's exact test P-value from iMKT

## How to apply

Retrieve genome variation data and ancestral-state information from 1000GP Phase III, plus human–chimpanzee divergence, from PopHuman in VCF format, applying the 1000GP pilot mask to exclude low-quality variants. Build unfolded site frequency spectrum and divergence count tables per gene using the python/bash human pipeline (Jupyter notebook 02_Human pipeline from iMKTData), classifying sites into functional classes (0-fold = selected class; 4-fold = neutral class). Stratify genes into a candidate set (e.g. high-recombination bins defined with Bhèrer et al. 2017 recombination rate data) and a genome-wide comparison set. Run the desired MK-derived test (standard MKT, FWWMKT, eMKT, or aMKT) on each set with the iMKT R package, passing the daf data frame (DAF, Pi, P0) and divergence data frame (mi, Di, m0, D0), then tabulate α (and ω, ωA, ωD) per population and gene group for comparison.

## Related tools

- **iMKT** (Web-based service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT) on pre-loaded human (PopHuman) and Drosophila (PopFly) data or user data) — https://imkt.uab.cat
- **iMKT (R package)** (R functions (e.g. standardMKT, eMKT) that take daf and divergence data frames and return α plus Fisher's exact test P-value and MKT/omega tables) — https://github.com/BGD-UAB/iMKT
- **Python** (Used in the 02_Human pipeline Jupyter notebook to extract unfolded SFS and divergence counts from 1000GP VCFs) — https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- **bash** (Raw scripts in src/ of the iMKTData repo used to execute/parallelize the human pipeline steps) — https://github.com/BGD-UAB/iMKTData
- **Jupyter** (Notebook (02_Human pipeline, Python 3.6 kernel) running the human pipeline step by step) — https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- The output file (e.g. emkt_high_recombination_ceu_yri_results.tsv) exists and contains alpha estimates for both CEU and YRI under both the high-recombination and whole-genome gene sets
- The analysis reproduces the reported pattern of predominantly negative α values in most cases, with visible differences between the high-recombination and whole-genome gene groups
- MKT count tables include unfolded SFS, analyzable sites by largest transcript, and divergence split by 0-fold (selected) and 4-fold (neutral) functional classes
- iMKT function output contains the expected components: alpha.symbol, Fisher's exact test P-value, MKT table (polymorphism/divergence per class), and divergence metrics Ka, Ks, omega, omegaA, omegaD
- Low-quality variants were removed using the 1000GP pilot accessible-genome mask before count table construction

## Limitations

- Pipelines require downloading external data (1000GP Phase III VCFs from PopHuman, Bhèrer et al. 2017 recombination data, pilot mask) with paths that need to be changed locally; multiple steps must be parallelized manually via custom bash scripts
- The iMKTData pipelines were developed and validated on a conda environment (imktData.yml) on a local server with 100GB RAM and 16 Intel Xeon CPUs, so resource needs are non-trivial
- eMKT/standard MKT estimates of α can be negative, as observed for human CEU and YRI comparisons; interpret negative α as lack of detectable adaptive evolution rather than a failure

## Evidence

- [readme] Genome variation data and information of the ancestral state of the variants generated by the 1000GP Phase III: "Genome variation data and information of the ancestral state of the variants generated by the 1000GP Phase III (1000 Genomes Project Consortium 2015), together with divergence between humans and"
- [methods] Pilot mask usage: "[Pilot mask](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/supporting/accessible_genome_masks) to exclude low quality variants download from 1000GP ftp"
- [intro] iMKT performs four MKT types: "Here we present iMKT (acronym for integrative McDonald and Kreitman test), a web-based service performing the four MKT types described in Appendix 1 and Figure 1."
- [readme] iMKT package usage and inputs: [evidence span withheld — anchor status: not_found]
- [intro] Comparing user gene sets with genome-wide distribution: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [readme] Pipeline outputs description: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes"
