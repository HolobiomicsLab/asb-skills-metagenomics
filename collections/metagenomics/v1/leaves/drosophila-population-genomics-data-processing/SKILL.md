---
name: drosophila-population-genomics-data-processing
description: Use when raw D. melanogaster DGN/PopFly FASTA alignments and divergence
  data must be converted into per-gene DAF spectra (0-fold/4-fold classes) and divergence
  counts for MKT-based estimation of alpha.
when_to_use_negative:
- 'Human data: use the 02_Human pipeline (1000GP Phase III VCF from PopHuman with
  ancestral state and pilot mask) instead of the D. melanogaster FASTA pipeline'
- You already have per-gene DAF/divergence tables — skip processing and call iMKT
  (R package or web service) directly on them
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: Python
  role: Executes the 01_D. melanogaster pipeline (dmelProteins.ipynb) that builds
    DAF and divergence tables from PopFly FASTA data
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- name: bash
  role: Raw src/ scripts executed by the notebook; parallelizable pipeline steps
  repo: https://github.com/BGD-UAB/iMKTData
- name: Jupyter
  role: Notebook environment (Python 3.6 kernel) for step-by-step execution of the
    D. melanogaster pipeline
  repo: https://github.com/jmurga/iMKTData
- name: iMKT (R package)
  role: Consumes the generated daf and divergence data frames to compute alpha via
    standardMKT, eMKT and other MK-derived methods
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web service alternative for analyzing the generated tables, including 'Compare
    against whole-genome distribution' analyses
  repo: https://imkt.uab.cat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:dd54f6ec9b487fbdd5703c38a178705b0b4a7eb4e4806c0bf31083051cd062af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# drosophila-population-genomics-data-processing

## Summary

Processes Drosophila melanogaster population genomic variation (Drosophila Genome Nexus alignments from PopFly, FASTA) plus D. melanogaster–D. simulans divergence data and Comeron et al. 2012 recombination rates into unfolded site frequency spectrum (DAF) and divergence tables per gene, analyzable by the iMKT web service or R package.

## When to use

Use when raw D. melanogaster DGN/PopFly FASTA alignments and divergence data must be converted into per-gene DAF spectra (0-fold/4-fold classes) and divergence counts for MKT-based estimation of alpha.

## When NOT to use

- Human data: use the 02_Human pipeline (1000GP Phase III VCF from PopHuman with ancestral state and pilot mask) instead of the D. melanogaster FASTA pipeline
- You already have per-gene DAF/divergence tables — skip processing and call iMKT (R package or web service) directly on them

## Inputs

- Drosophila Genome Nexus variation alignments retrieved from PopFly in FASTA format
- D. melanogaster–D. simulans divergence data retrieved from PopFly in FASTA format
- Comeron et al. 2012 recombination map
- dmelProteins.ipynb Jupyter notebook (Python 3.6 kernel) and src/ bash scripts from github.com/BGD-UAB/iMKTData

## Outputs

- Tab-delimited file with unfolded site frequency spectrum (DAF, Pi, P0), analyzable sites by largest transcript, and divergence (mi, Di, m0, D0) by gene, population, and MKT functional class (0-fold selected; 4-fold neutral)
- Per-gene DAF/divergence tables formatted for the iMKT web service and iMKT R package (e.g. emkt_alpha_results.tsv)

## How to apply

Run the 01_D. melanogaster pipeline in the dmelProteins Jupyter notebook (Python 3.6/bash) from the iMKTData repository, within the conda environment imktData.yml on a machine with ~100GB RAM and 16 cores. Download DGN variation data and D. melanogaster–D. simulans divergence data from PopFly in FASTA format (also available at johnpool.net/genomes.html) and the Comeron et al. 2012 recombination map, adjusting file paths as needed; steps can be parallelized with custom bash scripts. The pipeline classifies sites by MKT functional class (0-fold as the selected class, 4-fold as the neutral class), computes the unfolded site frequency spectrum, and emits a tab-delimited file with DAF, polymorphism (Pi, P0), analyzable sites by largest transcript, and divergence (mi, Di, m0, D0) by gene and population. These tables are then directly consumable by the iMKT web service or iMKT R package (devtools::install_github('BGD-UAB/iMKT')) to run standard MKT, eMKT and other variants, e.g. comparing a high-recombination gene set (>7 cM/Mb) against the whole-genome distribution.

## Related tools

- **Python** (Executes the 01_D. melanogaster pipeline (dmelProteins.ipynb) that builds DAF and divergence tables from PopFly FASTA data) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- **bash** (Raw src/ scripts executed by the notebook; parallelizable pipeline steps) — https://github.com/BGD-UAB/iMKTData
- **Jupyter** (Notebook environment (Python 3.6 kernel) for step-by-step execution of the D. melanogaster pipeline) — https://github.com/jmurga/iMKTData
- **iMKT (R package)** (Consumes the generated daf and divergence data frames to compute alpha via standardMKT, eMKT and other MK-derived methods) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web service alternative for analyzing the generated tables, including 'Compare against whole-genome distribution' analyses) — https://imkt.uab.cat

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Output file is tab-delimited and contains DAF, Pi, P0, mi, Di, m0, D0 columns partitioned by 0-fold (selected) and 4-fold (neutral) functional classes
- Downstream eMKT on the resulting tables reproduces published estimates, e.g. mean alpha 0.602 (SD 0.032) for 475 high-recombination genes (>7 cM/Mb) vs 0.44 (SD 0.055) for all 13,753 RAL genes
- Input validation: variation and divergence data must both originate from PopFly FASTA (DGN / D. melanogaster–D. simulans) and recombination rates from Comeron et al. 2012
- The pipeline runs to completion in the imktData.yml conda environment without path errors, since README notes paths need to be changed per installation
- Gene counts and population labels in output match the analyzed RAL dataset (13,753 protein-coding genes genome-wide)

## Limitations

- Pipeline requires downloading external files and manually changing paths in the notebook; multiple steps must be parallelized by the user via custom bash scripts
- Developed/tested in a local conda environment (imktData.yml) on a server with 100GB RAM and 16 Intel Xeon CPUs — smaller machines may struggle with whole-genome FASTA processing
- Human analysis requires a different pipeline (PopHuman VCF plus 1000GP pilot mask), so this skill's outputs are specific to the D. melanogaster FASTA workflow

## Evidence

- [readme] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [readme] Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes"
- [readme] Pipeline were developed in the conda enviroment *imktData.yml* in local server: 100GB RAM and 16 Intel(R) Xeon(R) CPU.: "Pipeline were developed in the conda enviroment *imktData.yml* in local server: 100GB RAM and 16 Intel(R) Xeon(R) CPU."
- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [methods] Recomb data from Comeron et al. 2012: "Recomb data from Comeron et al. 2012"
