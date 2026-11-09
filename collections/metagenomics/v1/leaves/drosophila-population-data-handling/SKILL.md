---
name: drosophila-population-data-handling
description: Use when estimating alpha or selection regimes (adaptive, neutral, strongly/weakly
  deleterious) for D. melanogaster protein-coding genes from polymorphism/divergence
  data, e.g. comparing gene sets such as high-recombination regions against the whole
  genome.
when_to_use_negative:
- Your data are human population genomic data (1000GP Phase III VCF via PopHuman with
  Bhèrer et al. 2012 recombination and the pilot mask) — use the human pipeline instead
  of the D. melanogaster pipeline
- You need raw variant calling or genome assembly rather than MK-test-ready polymorphism/divergence
  tables — iMKT expects pre-computed daf and divergence data frames
- Only single-gene, single-population alpha estimation is needed with no gene-set
  comparison — the web service front-end may suffice without running the full data
  pipeline
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: iMKT (R package)
  role: Computes the four MK-derived tests (standard MKT, FWWMKT, eMKT, aMKT) on the
    daf and divergence data frames to estimate alpha and selection regimes; installed
    via devtools::install_github("BGD-UAB/iMKT")
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web-based service alternative for running the four MKT types on pre-loaded
    D. melanogaster protein-coding gene sequences or user data
  repo: https://imkt.uab.cat
- name: Python
  role: Used in the 01_D. melanogaster pipeline notebook (dmelProteins.ipynb) to extract
    the unfolded SFS and divergence tables from PopFly FASTA alignments
  repo: https://github.com/jmurga/iMKTData
- name: bash
  role: Pipeline scripting language for the D. melanogaster data extraction steps
    in the Jupyter notebooks
- name: Jupyter
  role: Hosts the step-by-step notebooks (dmelProteins.ipynb) executing the pipeline
    on a Python 3.6 kernel
  repo: https://github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:c31e286016b7c8f0cb0c30042fe8c9afb623582bb759462acf96b48c3f5e1196
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGD-UAB/iMKT
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# drosophila-population-data-handling

## Summary

Prepare and analyze D. melanogaster population genomic data (DGN alignments via PopFly, divergence vs D. simulans, Comeron et al. 2012 recombination maps) as unfolded site frequency spectrum tables suitable for MK-derived analyses with the iMKT web service or iMKT R package.

## When to use

Use when estimating alpha or selection regimes (adaptive, neutral, strongly/weakly deleterious) for D. melanogaster protein-coding genes from polymorphism/divergence data, e.g. comparing gene sets such as high-recombination regions against the whole genome.

## When NOT to use

- Your data are human population genomic data (1000GP Phase III VCF via PopHuman with Bhèrer et al. 2012 recombination and the pilot mask) — use the human pipeline instead of the D. melanogaster pipeline
- You need raw variant calling or genome assembly rather than MK-test-ready polymorphism/divergence tables — iMKT expects pre-computed daf and divergence data frames
- Only single-gene, single-population alpha estimation is needed with no gene-set comparison — the web service front-end may suffice without running the full data pipeline

## Inputs

- Drosophila Genome Nexus (DGN) multiple sequence alignments in FASTA format retrieved from PopFly
- D. melanogaster vs D. simulans divergence data from PopFly
- Comeron et al. 2012 recombination map data
- daf data frame with columns daf, Pi, P0 and divergence data frame with columns mi, Di, m0, D0

## Outputs

- Tab-delimited file of unfolded site frequency spectrum, analyzable sites by largest transcript, and divergence per gene, population, and MKT functional class (0-fold/4-fold)
- Per-gene or per-gene-set estimates of alpha (and omega, omegaA, omegaD, Fisher's exact test P-value) from the chosen MK-derived method
- Summary statistics (mean and SD of alpha) per gene group, e.g. results_zi_emkt_recombination.tsv

## How to apply

Retrieve D. melanogaster variation data from the Drosophila Genome Nexus together with divergence data against D. simulans from PopFly in FASTA format, and recombination data from Comeron et al. 2012. Run the python/bash D. melanogaster pipeline (Jupyter notebook dmelProteins.ipynb) to produce a tab-delimited file with the unfolded site frequency spectrum, analyzable sites by largest transcript, and divergence per gene, population, and MKT functional class (0-fold as selected class, 4-fold as neutral class). Load the resulting daf and divergence data frames into the iMKT R package (installed via devtools from GitHub BGD-UAB/iMKT) and apply the MK-derived method of choice (standardMKT, FWWMKT, eMKT, aMKT), which takes the daf data frame (DAF, Pi, P0) and divergence data frame (mi, Di, m0, D0) as inputs. Partition genes by the Comeron et al. 2012 recombination map or other user criteria (e.g. chromosome localization) to compare a selected gene set against the genome-wide distribution; for the ZI population the eMKT comparison of high-recombination genes yields mean alpha 0.633 (SD 0.028) vs 0.457 (SD 0.053) for all 13,753 genes.

## Related tools

- **iMKT (R package)** (Computes the four MK-derived tests (standard MKT, FWWMKT, eMKT, aMKT) on the daf and divergence data frames to estimate alpha and selection regimes; installed via devtools::install_github("BGD-UAB/iMKT")) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web-based service alternative for running the four MKT types on pre-loaded D. melanogaster protein-coding gene sequences or user data) — https://imkt.uab.cat
- **Python** (Used in the 01_D. melanogaster pipeline notebook (dmelProteins.ipynb) to extract the unfolded SFS and divergence tables from PopFly FASTA alignments) — https://github.com/jmurga/iMKTData
- **bash** (Pipeline scripting language for the D. melanogaster data extraction steps in the Jupyter notebooks)
- **Jupyter** (Hosts the step-by-step notebooks (dmelProteins.ipynb) executing the pipeline on a Python 3.6 kernel) — https://github.com/jmurga/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- The pipeline output is a tab-delimited file containing unfolded SFS, analyzable sites by largest transcript, and divergence per gene, population, and MKT functional class (0-fold selected, 4-fold neutral)
- A standardMKT sanity check on the bundled myDafData/myDivergenceData returns alpha ≈ 0.2364499 and Fisher's exact test P-value ≈ 1.48e-183
- Reproducing the ZI high-recombination comparison must give mean alpha 0.633 (SD 0.028) for high-recombination genes and 0.457 (SD 0.053) for all 13,753 D. melanogaster genes
- The gene set partitioning must cover the full 13,753 protein-coding genes with the Comeron et al. 2012 map defining the high-recombination subset
- Group means must be written with matching summary statistics to the results file (e.g. results_zi_emkt_recombination.tsv) before the task is accepted

## Limitations

- Pipelines depend on external downloads (PopFly FASTA data, DGN site) whose paths must be manually changed, and parallel steps require custom bash scripts run on the server
- The reference pipeline environment was a conda environment (imktData.yml) on a local server with 100GB RAM and 16 Intel Xeon CPUs; smaller machines may struggle
- iMKT's four MK-derived methods behave differently according to data properties, so method choice (standardMKT vs FWWMKT vs eMKT vs aMKT) affects the estimated alpha

## Evidence

- [methods] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [readme] Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes"
- [readme] most functions require two input parameters: daf (data frame containing DAF, Pi and P0) and divergence (data frame containing mi, Di, m0, D0) and return the estimation of alpha: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline were developed in the conda enviroment imktData.yml in local server: 100GB RAM and 16 Intel(R) Xeon(R) CPU: "Pipeline were developed in the conda enviroment *imktData.yml* in local server: 100GB RAM and 16 Intel(R) Xeon(R) CPU."
