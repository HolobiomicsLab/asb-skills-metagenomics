---
name: recombination-rate-stratified-gene-comparison
description: Use when testing complex hypotheses that require comparing selective
  regimes (α, adaptive/neutral/deleterious fractions) of a user-selected gene set
  — e.g. defined by recombination rate bins or chromosome localization — against the
  whole-genome distribution.
when_to_use_negative:
- Input is raw sequencing data (e.g. unprocessed VCF/FASTA) rather than per-gene MKT
  count tables — the data-retrieval/preprocessing pipeline skill should run first
- The question concerns a single gene or a non-genic region without a defined gene
  set to contrast against the genome-wide distribution
- The organism or population has no compatible pre-loaded PopFly/PopHuman data or
  user-supplied count tables in iMKT's expected daf/divergence format
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iMKT
  role: Web-based service performing the four MKT types (standard MKT, FWWMKT, eMKT,
    aMKT) on pre-loaded human and D. melanogaster gene sets or user data, with advanced
    options for comparing a selected gene set to the genome-wide distribution
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: 'Programmatic execution of MK-derived tests: functions take daf and divergence
    data frames and return α estimation per recombination-stratified gene set'
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Used in the human (02) pipeline Jupyter notebook to extract unfolded site
    frequency spectra and divergence counts from PopHuman VCF per gene and population
  repo: https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- name: bash
  role: Used alongside Python in the human pipeline scripts to parallelize count-table
    generation steps
  repo: https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- name: Jupyter
  role: Notebooks (Python 3.6 kernel) that execute the pipeline step by step to generate
    the tab-delimited count tables consumed by iMKT
  repo: https://github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:7dab15bf8b059e23982614f25b6dc33e80b9f00cea62a97a0fb4d61afa610f96
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# recombination-rate-stratified-gene-comparison

## Summary

Stratifies protein-coding genes into a high-recombination set versus a genome-wide set using published recombination maps (Bhèrer et al. 2017 for human, Comeron et al. 2012 for Drosophila), then runs MK-derived tests (e.g. eMKT) per stratum via iMKT to test whether genes in high-recombination regions show different rates of adaptation (α).

## When to use

Use when testing complex hypotheses that require comparing selective regimes (α, adaptive/neutral/deleterious fractions) of a user-selected gene set — e.g. defined by recombination rate bins or chromosome localization — against the whole-genome distribution.

## When NOT to use

- Input is raw sequencing data (e.g. unprocessed VCF/FASTA) rather than per-gene MKT count tables — the data-retrieval/preprocessing pipeline skill should run first
- The question concerns a single gene or a non-genic region without a defined gene set to contrast against the genome-wide distribution
- The organism or population has no compatible pre-loaded PopFly/PopHuman data or user-supplied count tables in iMKT's expected daf/divergence format

## Inputs

- Pre-calculated MKT count tables (unfolded site frequency spectrum, analyzed sites, divergence per gene, population, and functional class: 0-fold neutral, 4-fold selected) in tab-delimited format
- Human: 1000GP Phase III VCF variation data and human–chimpanzee divergence from PopHuman, with ancestral-state information
- Human recombination rate data (Bhèrer et al. 2017) or Drosophila recombination data (Comeron et al. 2012) for gene stratification
- 1000GP pilot mask (accessible genome masks) to exclude low-quality variants
- iMKT R package `daf` data frame (daf, Pi, P0) and `divergence` data frame (mi, Di, m0, D0)

## Outputs

- Alpha (α) estimates and selection-regime fractions (d: strongly deleterious, b: weakly deleterious, f: neutral) per gene set and population
- Fisher's exact test P-values and MKT contingency tables per analysis
- Divergence metrics per stratum: Ka, Ks, omega, omegaA, omegaD
- e.g. a summary table such as emkt_high_recombination_ceu_yri_results.tsv with α for CEU and YRI under high-recombination and whole-genome gene sets

## How to apply

First retrieve pre-calculated MKT count tables: for human, 1000GP Phase III variation plus human–chimpanzee divergence from PopHuman in VCF format, with recombination bins from Bhèrer et al. 2017; for Drosophila, DGN variation and D. melanogaster–D. simulans divergence from PopFly in FASTA with Comeron et al. 2012 recombination data. Stratify genes into a high-recombination set and a whole-genome set according to the recombination map. Run the MK-derived test of interest (standard MKT, FWWMKT, eMKT, or aMKT) through the iMKT web service or the iMKT R package on each gene set separately — for humans, per population such as CEU and YRI. The R package functions take a `daf` data frame (DAF, Pi, P0) and a `divergence` data frame (mi, Di, m0, D0) and return α plus the Fisher's exact test P-value and divergence metrics (Ka, Ks, omega, omegaA, omegaD). Tabulate α per population and gene group and compare the strata; decide whether high-recombination genes differ from the genome-wide distribution. For human analysis, apply the 1000GP pilot mask to exclude low-quality variants before counting polymorphism.

## Related tools

- **iMKT** (Web-based service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT) on pre-loaded human and D. melanogaster gene sets or user data, with advanced options for comparing a selected gene set to the genome-wide distribution) — https://imkt.uab.cat
- **iMKT (R package)** (Programmatic execution of MK-derived tests: functions take daf and divergence data frames and return α estimation per recombination-stratified gene set) — https://github.com/BGD-UAB/iMKT
- **Python** (Used in the human (02) pipeline Jupyter notebook to extract unfolded site frequency spectra and divergence counts from PopHuman VCF per gene and population) — https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- **bash** (Used alongside Python in the human pipeline scripts to parallelize count-table generation steps) — https://github.com/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- **Jupyter** (Notebooks (Python 3.6 kernel) that execute the pipeline step by step to generate the tab-delimited count tables consumed by iMKT) — https://github.com/jmurga/iMKTData

## Examples

```
standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Output file (e.g. emkt_high_recombination_ceu_yri_results.tsv) exists and contains α estimates for both CEU and YRI under both the high-recombination and whole-genome gene sets
- The reported pattern matches expectations: predominantly negative α values in the human analysis with visible differences between the high-recombination and whole-genome gene groups
- Every iMKT run reports α together with a Fisher's exact test P-value and a complete MKT table (neutral and selected class polymorphism/divergence counts)
- Count tables include unfolded site frequency spectrum, analyzable sites by largest transcript, and divergence by genes, populations, and MKT functional classes (0-fold neutral, 4-fold selected)
- Low-quality variants were excluded via the 1000GP pilot mask before computing human polymorphism counts

## Limitations

- α estimates can be negative (as seen for CEU and YRI), which complicates direct biological interpretation of 'rate of adaptation'
- Pipelines were developed and tested on a specific environment (conda env imktData.yml, 100 GB RAM, 16 Intel Xeon CPUs); the repository notes multiple steps need custom parallelization or manual server execution and that download paths must be changed by the user
- Different MKT types behave differently on the same data, so results (α) are methodology-dependent; iMKT is designed to allow simultaneous comparison of all four MKTs

## Evidence

- [abstract] Advanced options allow testing complex hypotheses, illustrated by an application example on whether genes in high recombination regions undergo higher rates of adaptation: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [intro] iMKT allows comparing a user-selected gene set with the genome-wide distribution: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [readme] Human data sources: PopHuman VCF, 1000GP Phase III, Bhèrer et al. 2017 recombination, pilot mask: "Genome variation data and information of the ancestral state of the variants generated by 1000GP Phase III"
- [readme] Pipeline output format: tab-delimited count tables analyzable by iMKT: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes"
- [readme] iMKT R package function signature and example output: [evidence span withheld — anchor status: not_found]
