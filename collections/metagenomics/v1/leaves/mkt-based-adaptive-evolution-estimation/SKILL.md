---
name: mkt-based-adaptive-evolution-estimation
description: Use when you have unfolded site frequency spectrum (DAF) data plus divergence
  counts for neutral (4-fold/0) and selected (0-fold/i) classes per gene and need
  to estimate α, or to compare α of a user-selected gene set (e.g. high-recombination
  bins) against the genome-wide distribution.
when_to_use_negative:
- Input data are raw VCF/FASTA rather than the iMKTData-derived DAF and divergence
  count tables — first run the iMKTData python/bash pipelines to unfold the site frequency
  spectrum
- Your question requires inferring selection on non-coding or non-protein-coding sequence,
  whereas iMKT targets protein-coding DNA (0-fold/4-fold functional classes)
- You need per-variant selective inference rather than gene/gene-set level α estimates;
  the MKT framework outputs aggregate selection regime fractions (α, b, d, f) per
  unit
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0199
tools:
- name: iMKT
  role: Web service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT)
    on user data or pre-loaded PopFly/PopHuman gene sets
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: R functions (e.g. standardMKT, FWWMKT, eMKT, aMKT) taking daf and divergence
    data frames to compute α and selection regime fractions locally
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKTData pipelines
  role: Python/bash Jupyter pipelines (01 D. melanogaster, 02 Human) that generate
    the unfolded SFS and divergence-by-gene tab-delimited inputs consumed by iMKT
  repo: https://github.com/BGD-UAB/iMKTData
- name: PopHuman
  role: Source of 1000GP Phase III VCF variation data with ancestral states and human-chimpanzee
    divergence per gene
- name: PopFly
  role: Source of DGN variation alignments and D. melanogaster–D. simulans divergence
    data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:c309cd0db31e22ae5b3b53926ee821e491b1e8fbc58c6915d5035075530d1845
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PopFly
  - PopHuman
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  - iMKTData pipelines
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# mkt-based-adaptive-evolution-estimation

## Summary

Estimates adaptive evolution (α) and other selection regimes using the McDonald and Kreitman test family (standard MKT, FWWMKT, eMKT, aMKT) on polymorphism and divergence data, e.g. via the iMKT R package or web service on PopFly/PopHuman-derived DAF and divergence tables.

## When to use

Use when you have unfolded site frequency spectrum (DAF) data plus divergence counts for neutral (4-fold/0) and selected (0-fold/i) classes per gene and need to estimate α, or to compare α of a user-selected gene set (e.g. high-recombination bins) against the genome-wide distribution.

## When NOT to use

- Input data are raw VCF/FASTA rather than the iMKTData-derived DAF and divergence count tables — first run the iMKTData python/bash pipelines to unfold the site frequency spectrum
- Your question requires inferring selection on non-coding or non-protein-coding sequence, whereas iMKT targets protein-coding DNA (0-fold/4-fold functional classes)
- You need per-variant selective inference rather than gene/gene-set level α estimates; the MKT framework outputs aggregate selection regime fractions (α, b, d, f) per unit

## Inputs

- DAF table (tab-delimited; columns daf, Pi, P0) per population and functional class from the iMKTData pipelines
- Divergence table (tab-delimited; columns mi, Di, m0, D0) per gene/population
- PopHuman VCF-derived human variation data (1000GP Phase III with ancestral state) and human-chimpanzee divergence
- PopFly/DGN FASTA alignments for D. melanogaster vs D. simulans (optional Drosophila use)
- Recombination rate data (Bhèrer et al. 2017 for human; Comeron et al. 2012 for Drosophila) for gene-set stratification
- 1000GP pilot mask (accessible genome masks) to exclude low-quality variants

## Outputs

- α (alpha) estimate per population/gene set for the chosen MK-derived method
- Fisher's exact test P-value and MK table (polymorphism/divergence per neutral and selected class)
- Divergence metrics: Ka, Ks, omega, omegaA, omegaD
- e.g. emkt_high_recombination_ceu_yri_results.tsv with α for CEU and YRI under high-recombination and whole-genome gene sets

## How to apply

Prepare two input data frames per analysis: a 'daf' table with columns daf, Pi, P0 (polymorphism by derived allele frequency category) and a 'divergence' table with mi, Di, m0, D0 (analyzed sites and divergent sites for selected and neutral classes), as produced by the iMKTData pipelines from 1000GP Phase III VCF (human, via PopHuman) or DGN FASTA (Drosophila, via PopFly). For humans, optionally stratify genes into a high-recombination set and a whole-genome set using Bhèrer et al. 2017 recombination data, then run the chosen MK-derived method (standardMKT, FWWMKT, eMKT, aMKT) on each set separately per population (e.g. CEU and YRI). iMKT returns α together with a Fisher's exact test P-value, an MK table, and divergence metrics (Ka, Ks, omega, omegaA, omegaD); the eMKT corrects α for weakly deleterious variants, which is why estimates can be negative when slightly deleterious polymorphisms dominate. Decision point: choose the MKT variant according to how strongly segregation of weakly deleterious alleles biases the standard test; apply the 1000GP pilot mask to exclude low-quality variants before counting sites. Compare tabulated α estimates across populations and gene sets to test hypotheses such as whether genes in high-recombination regions undergo higher rates of adaptation.

## Related tools

- **iMKT** (Web service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT) on user data or pre-loaded PopFly/PopHuman gene sets) — https://imkt.uab.cat
- **iMKT (R package)** (R functions (e.g. standardMKT, FWWMKT, eMKT, aMKT) taking daf and divergence data frames to compute α and selection regime fractions locally) — https://github.com/BGD-UAB/iMKT
- **iMKTData pipelines** (Python/bash Jupyter pipelines (01 D. melanogaster, 02 Human) that generate the unfolded SFS and divergence-by-gene tab-delimited inputs consumed by iMKT) — https://github.com/BGD-UAB/iMKTData
- **PopHuman** (Source of 1000GP Phase III VCF variation data with ancestral states and human-chimpanzee divergence per gene)
- **PopFly** (Source of DGN variation alignments and D. melanogaster–D. simulans divergence data)

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Output file (e.g. emkt_high_recombination_ceu_yri_results.tsv) exists and contains α estimates for both CEU and YRI under both the high-recombination and whole-genome gene sets
- α values reproduce the reported pattern: predominantly negative α in human CEU/YRI comparisons, with visible differences between high-recombination and whole-genome gene groups
- Each α is accompanied by a Fisher's exact test P-value and a 2x2 MK table whose polymorphism/divergence counts match the input daf and divergence data frames
- Sanity-check divergence metrics: Ka, Ks, omega, omegaA, omegaD are finite and consistent (omega = Ka/Ks; standardMKT on package sample data gives alpha ≈ 0.236, Ka ≈ 0.021, Ks ≈ 0.085)
- Variant counts reflect application of the 1000GP pilot mask, so low-quality variants are excluded from the SFS before MKT computation

## Limitations

- Different MKT types behave differently on the same data; the standard MKT is biased by weakly deleterious variants, which can drive α negative, so method choice matters and iMKT's simultaneous comparison exists for this reason
- The pipelines that generate the inputs are resource-heavy (developed on a server with 100GB RAM and 16 Intel Xeon CPUs, conda environment imktData.yml) and multiple steps must be parallelized manually by the user
- Pre-loaded data are limited to D. melanogaster and human protein-coding genes; other organisms require user-supplied population genomic data
- Recombination-based gene-set comparisons depend on external recombination maps (Bhèrer et al. 2017 for human; Comeron et al. 2012 for Drosophila) whose resolution constrains the stratification

## Evidence

- [intro] It allows the detection and estimation of four selection regimes (adaptive, neutral, strongly deleterious and weakly deleterious) acting on protein-coding DNA sequences.: "It allows the detection and estimation of four selection regimes (adaptive, neutral, strongly deleterious and weakly deleterious) acting on protein-coding DNA sequences."
- [abstract] Advanced options allow testing complex hypotheses, illustrated by the high-recombination adaptation example.: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [methods] Genome variation data and divergence between humans and chimpanzees were retrieved from PopHuman in VCF; recombination data from Bhèrer et al. 2017.: "Genome variation data and information of the ancestral state of the variants generated by the 1000GP Phase III (1000 Genomes Project Consortium 2015), together with divergence between humans and"
- [intro] iMKT allows comparing the selective regimes of a user-selected gene set with the genome-wide distribution.: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [readme] iMKT is an R package to compute the McDonald and Kreitman test on polymorphism and divergence genomic data provided by the user or automatically downloaded from PopFly or PopHuman.: [evidence span withheld — anchor status: external_doc]
- [readme] Pipelines return a tab-delimited file with unfolded site frequency spectrum, analyzable sites and divergence by genes, populations and MKT functional classes (0-fold: selected; 4-fold: neutral).: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional"
