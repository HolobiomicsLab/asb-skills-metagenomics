---
name: population-genomic-data-retrieval
description: Use when building the polymorphism/divergence inputs for McDonald–Kreitman
  analyses, i.e. when DAF spectra and divergence counts must be pre-calculated from
  PopFly FASTA alignments or PopHuman 1000GP Phase III VCFs.
when_to_use_negative:
- The user already possesses pre-calculated SFS/divergence tables or plans to run
  iMKT on their own population genomic data rather than pre-loaded D. melanogaster/human
  genes
- The target organism is not D. melanogaster or human — the pipelines are specific
  to PopFly/DGN and PopHuman/1000GP Phase III data
- Only a standard MKT on user-supplied daf and divergence data frames is needed —
  the iMKT R package consumes those directly without any data retrieval
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3796
tools:
- name: Python
  role: Language of the dmelProteins.ipynb and humanProteins.ipynb pipelines that
    process FASTA/VCF inputs into SFS and divergence tables
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- name: bash
  role: Raw scripts in the iMKTData src/ folder used to execute and parallelize pipeline
    steps
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- name: Jupyter
  role: Notebooks folder contains the two main Jupyter Notebooks to execute the retrieval/pre-calculation
    pipeline step by step
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- name: iMKT
  role: 'Downstream consumer: the retrieved data enables on-the-fly MKT analyses via
    the iMKT web service and R package'
  repo: https://github.com/BGD-UAB/iMKT
- name: PopFly
  role: Source browser for D. melanogaster DGN variation and D. simulans divergence
    data in FASTA format
- name: PopHuman
  role: Source browser for 1000GP Phase III human VCFs with ancestral states and human–chimpanzee
    divergence data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:426f1cc0bac40fc9adf8c66f599300aaa5412194c38d7422741ea8b21500f6d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - PopFly
  - PopHuman
  - Python
  - bash
  - iMKT
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3796
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# population-genomic-data-retrieval

## Summary

Retrieves the population variation, divergence, recombination, and mask data required to pre-calculate unfolded 0-fold/4-fold SFS and divergence tables for iMKT analyses of D. melanogaster (DGN/PopFly) and human (1000GP Phase III/PopHuman) protein-coding genes.

## When to use

Use when building the polymorphism/divergence inputs for McDonald–Kreitman analyses, i.e. when DAF spectra and divergence counts must be pre-calculated from PopFly FASTA alignments or PopHuman 1000GP Phase III VCFs.

## When NOT to use

- The user already possesses pre-calculated SFS/divergence tables or plans to run iMKT on their own population genomic data rather than pre-loaded D. melanogaster/human genes
- The target organism is not D. melanogaster or human — the pipelines are specific to PopFly/DGN and PopHuman/1000GP Phase III data
- Only a standard MKT on user-supplied daf and divergence data frames is needed — the iMKT R package consumes those directly without any data retrieval

## Inputs

- D. melanogaster variation data from Drosophila Genome Nexus via PopFly, in FASTA format
- D. melanogaster vs D. simulans divergence alignments from PopFly (FASTA)
- Comeron et al. 2012 recombination map
- 1000GP Phase III human variation data with variant ancestral states from PopHuman, in VCF format
- Human–chimpanzee divergence data from PopHuman
- Bhèrer et al. 2017 recombination map
- 1000GP pilot mask (accessible genome masks) from the 1000GP ftp

## Outputs

- Tab-delimited per-gene, per-population unfolded 0-fold/4-fold SFS tables for 13,753 D. melanogaster genes across 16 populations
- Tab-delimited divergence tables (synonymous/nonsynonymous sites) by gene, population, and MKT functional class
- Tab-delimited unfolded SFS and divergence tables for 20,643 human genes across 26 populations
- Tables servable to the iMKT web service (https://imkt.uab.cat) and the iMKT R package

## How to apply

For D. melanogaster, download DGN variation data and D. melanogaster vs D. simulans divergence alignments from PopFly in FASTA format (also available at the DGN web site johnpool.net/genomes.html), plus the Comeron et al. 2012 recombination map, then run the dmelProteins.ipynb Jupyter notebook (python/bash) to produce per-gene, per-population unfolded SFS and divergence tables for 13,753 genes across 16 populations. For human, download 1000GP Phase III variation data with variant ancestral states together with human–chimpanzee divergence data from PopHuman in VCF format, the Bhèrer et al. 2017 recombination map, and the 1000GP pilot mask (accessible genome masks) from the 1000GP ftp; run humanProteins.ipynb applying the pilot mask to exclude low-quality variants, yielding unfolded SFS and divergence tables for 20,643 genes across 26 populations. Both notebooks run in a conda environment (imktData.yml, Python 3.6) on a 100GB RAM, 16-core server; hard-coded file paths must be adjusted so all downloads are read from local working directories. Output is tab-delimited files with unfolded site frequency spectrum, analyzable sites by largest transcript, and divergence by genes, populations, and MKT functional classes (0-fold: selected class; 4-fold: neutral class), consumable by the iMKT web service and R package.

## Related tools

- **Python** (Language of the dmelProteins.ipynb and humanProteins.ipynb pipelines that process FASTA/VCF inputs into SFS and divergence tables) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- **bash** (Raw scripts in the iMKTData src/ folder used to execute and parallelize pipeline steps) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/humanProteins.ipynb
- **Jupyter** (Notebooks folder contains the two main Jupyter Notebooks to execute the retrieval/pre-calculation pipeline step by step) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- **iMKT** (Downstream consumer: the retrieved data enables on-the-fly MKT analyses via the iMKT web service and R package) — https://github.com/BGD-UAB/iMKT
- **PopFly** (Source browser for D. melanogaster DGN variation and D. simulans divergence data in FASTA format)
- **PopHuman** (Source browser for 1000GP Phase III human VCFs with ancestral states and human–chimpanzee divergence data)

## Examples

```
jupyter notebook notebooks/dmelProteins.ipynb  # then notebooks/humanProteins.ipynb, after downloading PopFly FASTA / PopHuman VCF data and editing hard-coded paths
```

## Evaluation signals

- Generated SFS/divergence tables cover the full pre-calculation scope: 13,753 D. melanogaster genes across 16 populations and 20,643 human genes across 26 populations
- Output files are tab-delimited and include unfolded SFS, analyzable sites by largest transcript, and divergence by genes, populations, and 0-fold/4-fold functional classes
- Resulting tables match those served by the iMKT server
- Human pipeline variant set reflects application of the 1000GP pilot mask, i.e. low-quality variants excluded
- Pipelines executed within the imktData.yml conda environment (Python 3.6) after adjusting hard-coded file paths to local directories

## Limitations

- Hard-coded paths in the notebooks must be manually changed to local working directories before execution
- Only raw code is provided; parallelizable steps require the user to write custom bash scripts or run them manually on their server
- Pipelines were developed for a specific environment: conda imktData.yml on a local server with 100GB RAM and 16 Intel Xeon CPUs; smaller machines may struggle
- structure.sh in src/ overwrites the repository's notebook/ and src/ folders if executed

## Evidence

- [methods] DGN/PopFly retrieval: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [methods] Human data retrieval: "Genome variation data and information of the ancestral state of the variants generated by the 1000GP Phase III (1000 Genomes Project Consortium 2015)"
- [methods] Filter step: "Pilot mask to exclude low quality variants download from 1000GP ftp"
- [readme] Output format: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes"
- [readme] Execution environment: "Pipeline were developed in the conda enviroment *imktData.yml* in local server: 100GB RAM and 16 Intel(R) Xeon(R) CPU."
- [methods] Path adjustment caveat: "Pipelines execution requiere to download the following files. Paths would need to be changed too."
