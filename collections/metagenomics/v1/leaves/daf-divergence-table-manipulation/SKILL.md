---
name: daf-divergence-table-manipulation
description: Use when DGN/PopFly FASTA variation and D. melanogaster–D. simulans divergence
  data must be converted into tab-delimited DAF and divergence tables (0-fold/4-fold
  classes) for MKT-based alpha estimation or gene-set comparisons.
when_to_use_negative:
- Input is already an iMKT-format daf/divergence data frame — proceed directly to
  the iMKT functions instead of rebuilding tables
- Analysis targets human data — use the 02_Human pipeline (1000GP Phase III VCF from
  PopHuman, Bherer et al. 2017 recombination, pilot mask) rather than the D. melanogaster
  PopFly pipeline
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: iMKT (R package)
  role: Consumes the daf and divergence data frames to compute MK-derived tests and
    alpha; used with 'Compare against whole-genome distribution' for gene-set comparisons
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Runs the 01_D. melanogaster pipeline (dmelProteins.ipynb) that extracts unfolded
    SFS and divergence by genes
  repo: https://github.com/BGD-UAB/iMKTData
- name: bash
  role: Executes src/ raw scripts and custom parallelized steps of the D. melanogaster
    pipeline
  repo: https://github.com/BGD-UAB/iMKTData
- name: Jupyter
  role: Hosts the step-by-step Python 3.6 notebooks for data retrieve and pipeline
    execution
  repo: https://github.com/BGD-UAB/iMKTData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:30df4b3b52a4ab7fe4a0796bee87c9c50cc5d060c21469b6a140453bce6df441
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
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# daf-divergence-table-manipulation

## Summary

Build and manipulate unfolded site frequency spectrum (DAF) and divergence tables for D. melanogaster protein-coding genes so they can be analyzed with the iMKT web service or R package to estimate the adaptive substitution fraction (alpha).

## When to use

Use when DGN/PopFly FASTA variation and D. melanogaster–D. simulans divergence data must be converted into tab-delimited DAF and divergence tables (0-fold/4-fold classes) for MKT-based alpha estimation or gene-set comparisons.

## When NOT to use

- Input is already an iMKT-format daf/divergence data frame — proceed directly to the iMKT functions instead of rebuilding tables
- Analysis targets human data — use the 02_Human pipeline (1000GP Phase III VCF from PopHuman, Bherer et al. 2017 recombination, pilot mask) rather than the D. melanogaster PopFly pipeline

## Inputs

- Drosophila Genome Nexus (DGN) variation data in FASTA format, retrieved from PopFly or johnpool.net/genomes.html
- D. melanogaster–D. simulans divergence data from PopFly in FASTA format
- Comeron et al. 2012 recombination map (cM/Mb) for gene-set selection
- iMKTData dmelProteins.ipynb / src/ scripts and imktData.yml conda environment

## Outputs

- Tab-delimited DAF table with derived allele frequency spectrum (DAF, Pi, P0) per gene/population
- Tab-delimited divergence table with mi, Di, m0, D0 for 0-fold (selected) and 4-fold (neutral) classes
- Per-gene and gene-set alpha estimates and summary statistics (e.g., emkt_alpha_results.tsv)

## How to apply

Retrieve Drosophila Genome Nexus variation data plus D. melanogaster–D. simulans divergence from PopFly in FASTA format (also available from the DGN web site) and recombination data from Comeron et al. 2012. Run the Jupyter notebooks (Python 3.6 kernel) or src/ scripts from the iMKTData repository inside the conda environment imktData.yml — the authors used a local server with 100GB RAM and 16 Intel Xeon CPUs. Each pipeline returns a tab-delimited file containing the unfolded site frequency spectrum, analyzable sites by largest transcript, and divergence by genes, populations, and MKT functional classes (0-fold = selected class; 4-fold = neutral class). For gene-set hypotheses such as high-recombination genes (>7 cM/Mb from Comeron et al. 2012), subset genes and use the iMKT 'Compare against whole-genome distribution' option via the iMKT R package to obtain alpha estimates comparable with the genome-wide set of 13,753 RAL genes.

## Related tools

- **iMKT (R package)** (Consumes the daf and divergence data frames to compute MK-derived tests and alpha; used with 'Compare against whole-genome distribution' for gene-set comparisons) — https://github.com/BGD-UAB/iMKT
- **Python** (Runs the 01_D. melanogaster pipeline (dmelProteins.ipynb) that extracts unfolded SFS and divergence by genes) — https://github.com/BGD-UAB/iMKTData
- **bash** (Executes src/ raw scripts and custom parallelized steps of the D. melanogaster pipeline) — https://github.com/BGD-UAB/iMKTData
- **Jupyter** (Hosts the step-by-step Python 3.6 notebooks for data retrieve and pipeline execution) — https://github.com/BGD-UAB/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Reproduced workflow must recover the paper's gene-set result: high-recombination genes (>7 cM/Mb) mean alpha = 0.602 (SD 0.032) vs whole-genome 13,753 RAL genes mean alpha = 0.44 (SD 0.055)
- Output table must be tab-delimited and include unfolded SFS, analyzable sites by largest transcript, and divergence per gene/population split into 0-fold (selected) and 4-fold (neutral) classes
- iMKT functions on valid inputs return alpha plus a Fisher's exact test P-value and MKT table (e.g., standardMKT returns alpha.symbol = 0.2364499 on myDafData/myDivergenceData)
- Whole-genome count check: the RAL gene set should comprise all 13,753 protein-coding genes
- Pipelines must run in the imktData.yml conda environment on the documented resource scale (100GB RAM, 16 Xeon cores); deviations suggest missing downloads or wrong paths

## Limitations

- Pipeline execution requires downloading external files with paths that 'would need to be changed' manually; some steps must be parallelized with custom bash scripts
- The iMKTData repository provides only raw code for the main results — users must create their own folder structure (structure.sh overwrites notebooks/ and src/)
- Human and fly data differ in source and format: FASTA from PopFly/DGN vs VCF from PopHuman/1000GP Phase III, so pipelines are not interchangeable without adaptation

## Evidence

- [readme] Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes (0-fold: selected class; 4-fold: neutral class).: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional classes"
- [readme] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [intro] It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization)"
- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [readme] most functions require two input parameters: daf (data frame containing DAF, Pi and P0) and divergence (data frame containing mi, Di, m0, D0): [evidence span withheld — anchor status: not_found]
