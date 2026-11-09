---
name: reported-result-reproducibility-check
description: Use when a source card reports specific numeric findings (e.g. alpha
  mean 0.633 ± SD 0.028 for 475 high-recombination genes) that must be independently
  reproduced from pre-calculated polymorphism/divergence tables using a named tool.
when_to_use_negative:
- Input is raw population genomic data (FASTA/VCF alignments) rather than the pre-calculated
  polymorphism/divergence tables expected by iMKT — use the iMKTData pipelines first
- The target population or divergence species differs from D. melanogaster ZI vs D.
  simulans, since the reported alpha values being validated are specific to that comparison
- The goal is to test a novel hypothesis rather than verify already-reported numeric
  findings
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: iMKT (R package)
  role: Runs the eMKT (asymptotic MKT excluding weakly deleterious variants) on the
    gene sets and provides pre-loaded PopFly polymorphism/divergence data for D. melanogaster
    populations including ZI
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web-based service alternative that performs the four MKT types (standard MKT,
    FWW MKT, eMKT, aMKT) on pre-loaded D. melanogaster and human sequences
  repo: https://imkt.uab.cat
- name: iMKTData
  role: Source of the pipelines and Jupyter notebooks that generated the pre-calculated
    polymorphism/divergence tables from DGN alignments and Comeron et al. 2012 recombination
    data
  repo: github.com/BGD-UAB/iMKTData
- name: Python
  role: Used in the D. melanogaster pipeline notebook to extract unfolded site frequency
    spectra and divergence by genes
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- name: bash
  role: Raw scripts under src/ to execute pipeline steps, parallelizable via custom
    bash scripts
- name: Jupyter
  role: Notebooks running on Python 3.6 kernel to execute the D. melanogaster pipeline
    step by step
  repo: https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:28cea2cc62aa8899cb5e6ecab96eeaa1e4d57cb733fcfe4e464336a8585bc649
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
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reported-result-reproducibility-check

## Summary

Reproduces a published quantitative result — here, the eMKT alpha estimate for D. melanogaster genes in high-recombination regions vs the whole genome in the Zambia (ZI) population — by re-running the iMKT pipeline and comparing computed summary statistics against reported values before acceptance.

## When to use

Use when a source card reports specific numeric findings (e.g. alpha mean 0.633 ± SD 0.028 for 475 high-recombination genes) that must be independently reproduced from pre-calculated polymorphism/divergence tables using a named tool.

## When NOT to use

- Input is raw population genomic data (FASTA/VCF alignments) rather than the pre-calculated polymorphism/divergence tables expected by iMKT — use the iMKTData pipelines first
- The target population or divergence species differs from D. melanogaster ZI vs D. simulans, since the reported alpha values being validated are specific to that comparison
- The goal is to test a novel hypothesis rather than verify already-reported numeric findings

## Inputs

- Pre-calculated D. melanogaster ZI (Zambia) polymorphism/divergence tables shipped with the iMKT R package (unfolded SFS by DAF, Pi, P0 and divergence mi, Di, m0, D0 per gene)
- Comeron et al. 2012 recombination map for binning genes into high-recombination regions
- List of 13,753 D. melanogaster protein-coding genes

## Outputs

- Per-gene alpha estimates from eMKT for the high-recombination gene set (475 genes) and the full gene set (13,753 genes)
- Group-level summary statistics: mean and SD of alpha per gene group
- results_zi_emkt_recombination.tsv with per-group alpha statistics

## How to apply

Load the pre-calculated D. melanogaster polymorphism and divergence tables for the Zambia (ZI) population provided with the iMKT R package (data derived from the Drosophila Genome Nexus via PopFly and divergence to D. simulans). Partition the 13,753 protein-coding genes into a high-recombination set (475 genes, defined by the Comeron et al. 2012 recombination map) and the full genome-wide set. Run the eMKT (asymptotic MKT excluding weakly deleterious variants) implemented in iMKT on both gene sets to estimate alpha per gene, then compute mean and standard deviation of alpha per group. Validate by comparing against reported values — high-recombination mean alpha 0.633 (SD 0.028), genome-wide mean alpha 0.457 (SD 0.053) across 13,753 genes — before writing results_zi_emkt_recombination.tsv; the task is only accepted if the computed values reproduce the reported statistics.

## Related tools

- **iMKT (R package)** (Runs the eMKT (asymptotic MKT excluding weakly deleterious variants) on the gene sets and provides pre-loaded PopFly polymorphism/divergence data for D. melanogaster populations including ZI) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web-based service alternative that performs the four MKT types (standard MKT, FWW MKT, eMKT, aMKT) on pre-loaded D. melanogaster and human sequences) — https://imkt.uab.cat
- **iMKTData** (Source of the pipelines and Jupyter notebooks that generated the pre-calculated polymorphism/divergence tables from DGN alignments and Comeron et al. 2012 recombination data) — github.com/BGD-UAB/iMKTData
- **Python** (Used in the D. melanogaster pipeline notebook to extract unfolded site frequency spectra and divergence by genes) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb
- **bash** (Raw scripts under src/ to execute pipeline steps, parallelizable via custom bash scripts)
- **Jupyter** (Notebooks running on Python 3.6 kernel to execute the D. melanogaster pipeline step by step) — https://nbviewer.jupyter.org/github/jmurga/iMKTData/blob/master/notebooks/dmelProteins.ipynb

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); eMKT(highRecDaf, highRecDivergence, listCutoffs=TRUE) # compare group alpha stats to reported 0.633 (SD 0.028)
```

## Evaluation signals

- Computed mean alpha for the high-recombination gene group equals 0.633 with SD 0.028
- Computed mean alpha for the genome-wide group equals 0.457 with SD 0.053 over exactly 13,753 genes
- High-recombination group contains the expected 475 genes when partitioned using the Comeron et al. 2012 recombination map
- results_zi_emkt_recombination.tsv is produced with per-group alpha statistics matching the reported values
- eMKT outputs conform to the iMKT function structure (alpha estimate plus Fisher's exact test P-value and MKT table)

## Limitations

- iMKT requires pre-calculated polymorphism and divergence tables; generating them from raw DGN alignments via the iMKTData pipelines demands substantial resources (the original pipelines ran in a conda environment with 100GB RAM and 16 Intel Xeon CPUs)
- MKT-derived methods behave differently according to data properties — the standard MKT, FWW MKT, eMKT and aMKT give different alpha estimates, so validation only holds for the eMKT variant
- Recombination-based binning depends on the specific map used (Comeron et al. 2012); using a different recombination map will not reproduce the reported values

## Evidence

- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [readme] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [readme] Recomb data from Comeron et al. 2012: "Recomb data from [Comeron et al. 2012]"
- [intro] It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [supplementary] The corresponding R package is available for download at https://github.com/BGD-UAB/iMKT: "The corresponding R package is available for download at https://github.com/BGD-UAB/iMKT"
