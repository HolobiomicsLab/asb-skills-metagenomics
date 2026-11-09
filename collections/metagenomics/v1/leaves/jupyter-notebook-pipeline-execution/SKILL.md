---
name: jupyter-notebook-pipeline-execution
description: Use when raw population variation data (DGN FASTA alignments or 1000GP
  Phase III VCFs) must be converted into per-gene, per-population SFS and divergence
  tables for MKT analyses, and pre-computed server tables are insufficient.
when_to_use_negative:
- The user only needs an MKT on a small user-supplied polymorphism/divergence table
  — the iMKT R package functions (e.g. standardMKT) take daf and divergence data frames
  directly without running the full pipeline
- Pre-loaded tables on the iMKT server already cover the gene set and population of
  interest, making local recomputation redundant
- Data are not D. melanogaster DGN or human 1000GP Phase III protein-coding alignments
  — the notebooks contain hard-coded paths and species-specific processing logic
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: Jupyter
  role: Runs the dmelProteins.ipynb and humanProteins.ipynb notebooks step by step
    on a Python 3.6 kernel
- name: Python
  role: Core processing language for the D. melanogaster and human SFS/divergence
    pre-calculation pipelines
- name: bash
  role: Supports raw scripts in src/ and user-created custom parallelization scripts;
    structure.sh creates the working folder layout
- name: iMKT (R package)
  role: Downstream consumer of the tab-delimited DAF and divergence tables generated
    by the pipelines
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKTData
  role: Repository containing the two notebooks and src/ scripts that pre-calculate
    SFS and divergence tables
  repo: github.com/BGD-UAB/iMKTData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:9bfe14eb46023ab761861e3dc753253cd341905e03aea6da3c126410f66971b1
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
  - iMKTData
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# jupyter-notebook-pipeline-execution

## Summary

Execute step-by-step Jupyter notebooks (Python 3.6 kernel, python/bash cells) that pre-calculate unfolded site frequency spectra (0-fold/4-fold DAF) and divergence tables from population genomic data, as done in the iMKTData pipeline for D. melanogaster and human protein-coding genes.

## When to use

Use when raw population variation data (DGN FASTA alignments or 1000GP Phase III VCFs) must be converted into per-gene, per-population SFS and divergence tables for MKT analyses, and pre-computed server tables are insufficient.

## When NOT to use

- The user only needs an MKT on a small user-supplied polymorphism/divergence table — the iMKT R package functions (e.g. standardMKT) take daf and divergence data frames directly without running the full pipeline
- Pre-loaded tables on the iMKT server already cover the gene set and population of interest, making local recomputation redundant
- Data are not D. melanogaster DGN or human 1000GP Phase III protein-coding alignments — the notebooks contain hard-coded paths and species-specific processing logic

## Inputs

- DGN D. melanogaster population variation alignments with D. melanogaster vs D. simulans divergence data (FASTA, from PopFly or johnpool.net/genomes.html)
- Comeron et al. 2012 recombination map
- 1000GP Phase III human VCFs with variant ancestral states and human-chimpanzee divergence (VCF, from PopHuman)
- Bhèrer et al. 2017 recombination map
- 1000GP accessible-genome pilot mask (from 1000GP ftp)
- dmelProteins.ipynb and humanProteins.ipynb notebooks plus src/ scripts from github.com/jmurga/iMKTData

## Outputs

- Tab-delimited per-gene, per-population unfolded site frequency spectrum files for 0-fold and 4-fold degenerate sites
- Divergence tables (synonymous/nonsynonymous divergent and analyzable sites) by gene, population and MKT functional class
- Tables consumable by the iMKT web-service (https://imkt.uab.cat) and the iMKT R package as daf/divergence data frames

## How to apply

Download the required source files first: D. melanogaster variation and divergence data from PopFly in FASTA format plus the Comeron et al. 2012 recombination map, and human 1000GP Phase III VCF data with ancestral states plus human-chimpanzee divergence from PopHuman plus the Bhèrer et al. 2017 recombination map and the 1000GP accessible-genome pilot mask. Open the two iMKTData notebooks (dmelProteins.ipynb, humanProteins.ipynb) in the imktData.yml conda environment on a machine with ~100GB RAM and 16 CPUs, and adjust the hard-coded file paths so downloads are read from your local working directories. Run cells step by step, applying the pilot mask to exclude low-quality human variants, to produce unfolded 0-fold (selected) and 4-fold (neutral) SFS counts and divergence counts per gene and population. Output is a tab-delimited file with the unfolded SFS, analyzable sites by largest transcript, and divergence by gene, population and MKT functional class, serving 13,753 D. melanogaster genes across 16 populations and 20,643 human genes across 26 populations. The resulting tables can then be loaded into the iMKT web-service or R package as the daf and divergence data frames.

## Related tools

- **Jupyter** (Runs the dmelProteins.ipynb and humanProteins.ipynb notebooks step by step on a Python 3.6 kernel)
- **Python** (Core processing language for the D. melanogaster and human SFS/divergence pre-calculation pipelines)
- **bash** (Supports raw scripts in src/ and user-created custom parallelization scripts; structure.sh creates the working folder layout)
- **iMKT (R package)** (Downstream consumer of the tab-delimited DAF and divergence tables generated by the pipelines) — https://github.com/BGD-UAB/iMKT
- **iMKTData** (Repository containing the two notebooks and src/ scripts that pre-calculate SFS and divergence tables) — github.com/BGD-UAB/iMKTData

## Examples

```
jupyter notebook --execute notebooks/dmelProteins.ipynb  # run after editing hard-coded paths; human equivalent: notebooks/humanProteins.ipynb
```

## Evaluation signals

- Output tab-delimited files contain unfolded SFS counts, analyzable sites by largest transcript, and divergence by genes, populations and MKT functional classes (0-fold selected, 4-fold neutral)
- Coverage matches the pre-calculation scope: 13,753 D. melanogaster genes across 16 populations and 20,643 human genes across 26 populations
- Generated tables match the tables served by the iMKT server (imkt.uab.cat)
- Human variant tables reflect application of the 1000GP pilot mask (low-quality variants excluded)
- All notebooks execute in the imktData.yml conda environment without path errors after local path adjustments

## Limitations

- Hard-coded file paths in the notebooks must be manually changed to local working directories before execution
- Multiple pipeline steps must be manually parallelized — the user must write custom bash scripts or run steps manually on the server
- The reference deployment required ~100GB RAM and 16 Intel Xeon CPUs; smaller machines may fail
- structure.sh overwrites the notebook/ and src/ folders if executed, so repository versions must be restored afterwards

## Evidence

- [readme] Pipelines to extract unfolded site frequency spectrum from 1000GP VCF and DGN alignments.: "Pipelines to extract unfolded site frequency spectrum from 1000GP VCF and DGN alignments."
- [readme] Tab-delimited outputs: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes, populations and MKT functional"
- [readme] Conda environment and hardware: "Pipeline were developed in the conda enviroment *imktData.yml* in local server: 100GB RAM and 16 Intel(R) Xeon(R) CPU."
- [methods] Pilot mask filter: "[Pilot mask](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/supporting/accessible_genome_masks) to exclude low quality variants download from 1000GP ftp"
- [readme] Data download requirement: "Pipelines execution requiere to download the following files. Paths would need to be changed too."
