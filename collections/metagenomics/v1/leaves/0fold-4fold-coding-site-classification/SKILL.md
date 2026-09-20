---
name: 0fold-4fold-coding-site-classification
description: Use when you need per-gene, per-population unfolded 0-fold/4-fold SFS
  counts and divergence counts (mi, Di, m0, D0) from DGN FASTA alignments or 1000GP
  Phase III VCFs to run McDonald-Kreitman tests with iMKT.
when_to_use_negative:
- 'You already possess SFS and divergence tables (daf: daf/Pi/P0 and divergence: mi/Di/m0/D0
  data frames) — feed them directly to the iMKT R package or web service instead of
  re-running the pipelines'
- Your data is not D. melanogaster (DGN) or human (1000GP Phase III) protein-coding
  data; the pre-calculated pipelines cover only these two species
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_0622
tools:
- name: iMKTData
  role: Jupyter notebook pipelines (dmelProteins.ipynb, humanProteins.ipynb) that
    extract unfolded SFS and classify sites into 0-fold/4-fold MKT functional classes
  repo: https://github.com/BGD-UAB/iMKTData
- name: iMKT (R package)
  role: Downstream consumer of the daf and divergence data frames produced by this
    classification skill
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Implementation language of the dmelProteins.ipynb and humanProteins.ipynb
    pipelines
- name: Jupyter
  role: Execution environment for the step-by-step pipeline notebooks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:4877741143a9bfaacdfc6f29edccc516c8d9af1530c0db46f0c380ac33f97489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGD-UAB/iMKTData
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  - iMKTData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# 0fold-4fold-coding-site-classification

## Summary

Pre-processing pipelines (iMKTData Jupyter notebooks, Python/bash) that classify protein-coding sites into MKT functional classes — 0-fold (selected) and 4-fold (neutral) — to build unfolded site frequency spectrum (DAF) and divergence tables per gene and population, enabling on-the-fly MKT analyses in iMKT.

## When to use

Use when you need per-gene, per-population unfolded 0-fold/4-fold SFS counts and divergence counts (mi, Di, m0, D0) from DGN FASTA alignments or 1000GP Phase III VCFs to run McDonald-Kreitman tests with iMKT.

## When NOT to use

- You already possess SFS and divergence tables (daf: daf/Pi/P0 and divergence: mi/Di/m0/D0 data frames) — feed them directly to the iMKT R package or web service instead of re-running the pipelines
- Your data is not D. melanogaster (DGN) or human (1000GP Phase III) protein-coding data; the pre-calculated pipelines cover only these two species

## Inputs

- D. melanogaster population variation alignments from Drosophila Genome Nexus / PopFly (FASTA format)
- D. melanogaster vs D. simulans divergence alignments (PopFly FASTA)
- Comeron et al. 2012 recombination map
- Human 1000GP Phase III variation VCF with variant ancestral states (PopHuman)
- Human-chimpanzee divergence data (PopHuman VCF)
- Bhèrer et al. 2017 recombination map
- 1000GP accessible-genome pilot mask to exclude low-quality variants

## Outputs

- Tab-delimited unfolded DAF spectrum tables per gene and population (0-fold selected / 4-fold neutral classes)
- Tab-delimited divergence tables per gene and population (analyzable sites and divergent sites for each functional class)
- Coverage: 13,753 D. melanogaster genes across 16 populations; 20,643 human genes across 26 populations

## How to apply

Retrieve D. melanogaster variation data from the Drosophila Genome Nexus (via PopFly, FASTA format) with D. melanogaster vs D. simulans divergence alignments plus the Comeron et al. 2012 recombination map; retrieve human variation data with ancestral-state information from 1000GP Phase III (via PopHuman, VCF format), human-chimpanzee divergence, the Bhèrer et al. 2017 recombination map, and the 1000GP pilot mask to exclude low-quality variants. Run the dmelProteins.ipynb and humanProteins.ipynb Jupyter notebooks (Python 3.6/bash, conda environment imktData.yml on a 100GB RAM / 16 Xeon server) step by step, adjusting hard-coded file paths to your local working directories. The pipelines classify each site into 0-fold degenerate (selected class) or 4-fold degenerate (neutral class) and output tab-delimited files with unfolded DAF spectrum, analyzable sites by largest transcript, and divergence counts per gene, population, and functional class. Unfolding requires ancestral-state information, so variant polarisation must be preserved from the source data.

## Related tools

- **iMKTData** (Jupyter notebook pipelines (dmelProteins.ipynb, humanProteins.ipynb) that extract unfolded SFS and classify sites into 0-fold/4-fold MKT functional classes) — https://github.com/BGD-UAB/iMKTData
- **iMKT (R package)** (Downstream consumer of the daf and divergence data frames produced by this classification skill) — https://github.com/BGD-UAB/iMKT
- **Python** (Implementation language of the dmelProteins.ipynb and humanProteins.ipynb pipelines)
- **Jupyter** (Execution environment for the step-by-step pipeline notebooks)

## Examples

```
jupyter notebook notebooks/dmelProteins.ipynb
```

## Evaluation signals

- Output tables must cover the full pre-calculation scope: 13,753 D. melanogaster genes / 16 populations and 20,643 human genes / 26 populations
- Generated tab-delimited SFS and divergence tables must match the tables served by the iMKT server
- Divergence table columns must follow the iMKT schema (mi, Di, m0, D0) and DAF table columns (daf, Pi, P0) with 0-fold as selected class and 4-fold as neutral class
- Human variants must be filtered by the 1000GP pilot mask; low-quality variants outside the accessible genome should be absent from the SFS
- SFS must be unfolded (derived-allele counts), confirming ancestral-state information was applied during classification

## Limitations

- Hard-coded file paths in the notebooks must be manually adjusted for local execution; multiple steps need custom parallelization (bash scripts or manual server runs) since the repository ships only raw code
- Pipelines were developed in the conda environment imktData.yml on a local server with 100GB RAM and 16 Intel(R) Xeon(R) CPUs — smaller machines may fail
- Only 1000GP VCF and DGN alignment sources are supported; the human pipeline depends on the pilot mask, which restricts analysis to the accessible genome

## Evidence

- [readme] Pipelines to extract unfolded site frequency spectrum from 1000GP VCF and DGN alignments.: "Pipelines to extract unfolded site frequency spectrum from 1000GP VCF and DGN alignments."
- [readme] MKT functional classes (0-fold: selected class; 4-fold: neutral class): "divergence by genes, populations and MKT functional classes (0-fold: selected class; 4-fold: neutral class)"
- [methods] Drosophila Genome Nexus variation and D. simulans divergence retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [readme] Notebooks run step by step on Python 3.6: "notebooks/ folder include two main Jupyter Notebooks running on Python 3.6 kernel to execute step by step the pipeline."
- [readme] Pilot mask to exclude low quality variants: "[Pilot mask](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/supporting/accessible_genome_masks) to exclude low quality variants download from 1000GP ftp"
