---
name: genome-wide-alpha-distribution-comparison
description: Use when you need to test whether a biologically defined subset of protein-coding
  genes (e.g. by recombination rate bin or chromosome localization) shows a different
  alpha than the genome-wide gene set, using DAF and divergence tables.
when_to_use_negative:
- Your input is raw variant calls or VCF/FASTA alignments rather than pre-computed
  DAF/divergence tables — first run the iMKTData pipelines (dmelProteins notebook)
  to generate the SFS and divergence files
- You need to test a single gene or genomic region in isolation without a genome-wide
  comparison distribution
- Your data are human rather than D. melanogaster — use the humanProteins pipeline
  and PopHuman/1000GP-derived tables instead
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: iMKT (R package)
  role: Executes the eMKT with the 'Compare against whole-genome distribution' option
    on the selected gene set and the genome-wide set
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web-based service alternative for running the four MKT types and comparing
    selective regimes of a gene set against the genome-wide distribution
- name: Jupyter
  role: Hosts the dmelProteins python/bash notebook environment in which the iMKTData
    pipelines and analyses are executed
  repo: github.com/jmurga/iMKTData
- name: Python
  role: Used in the D. melanogaster pipeline notebook for data retrieval and building
    the DAF/divergence tables consumed by iMKT
  repo: github.com/jmurga/iMKTData
- name: bash
  role: Raw scripts in src/ of iMKTData used to execute pipeline steps for extracting
    unfolded SFS and divergence by gene
  repo: github.com/BGD-UAB/iMKTData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:8efbd869dfa978cbe56dc606a0b9c759c54602c975c897fdc6e4bc3a72890741
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
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genome-wide-alpha-distribution-comparison

## Summary

Runs the eMKT (via iMKT web service or R package) on a user-selected gene set using the 'Compare against whole-genome distribution' option, and compares the resulting adaptive substitution fraction (alpha) distribution against the genome-wide alpha distribution to test hypotheses such as whether genes in high-recombination regions undergo higher rates of adaptation.

## When to use

Use when you need to test whether a biologically defined subset of protein-coding genes (e.g. by recombination rate bin or chromosome localization) shows a different alpha than the genome-wide gene set, using DAF and divergence tables.

## When NOT to use

- Your input is raw variant calls or VCF/FASTA alignments rather than pre-computed DAF/divergence tables — first run the iMKTData pipelines (dmelProteins notebook) to generate the SFS and divergence files
- You need to test a single gene or genomic region in isolation without a genome-wide comparison distribution
- Your data are human rather than D. melanogaster — use the humanProteins pipeline and PopHuman/1000GP-derived tables instead

## Inputs

- Tab-delimited DAF table per gene (unfolded site frequency spectrum, Pi and P0 by DAF category) from the Drosophila Genome Nexus / PopFly
- Tab-delimited divergence table per gene (mi, Di, m0, D0 for 0-fold and 4-fold classes) from D. melanogaster–D. simulans divergence via PopFly
- Comeron et al. 2012 recombination map (cM/Mb per gene) used to define the high-recombination (>7 cM/Mb) gene set
- List of protein-coding genes defining the candidate subset (e.g. 475 high-recombination genes)

## Outputs

- Per-gene eMKT alpha estimates for the selected gene set and for the whole-genome set (13,753 genes)
- Summary statistics (mean and SD of alpha) per gene set, e.g. emkt_alpha_results.tsv with gene set, alpha estimates, and summary statistics

## How to apply

Load pre-calculated DAF (daf data frame with DAF, Pi, P0) and divergence (divergence data frame with mi, Di, m0, D0) tables derived from Drosophila Genome Nexus variation data and D. melanogaster–D. simulans divergence retrieved from PopFly in FASTA format. Define the gene subset using an explicit criterion — for the reference result, the 475 genes with recombination rate >7 cM/Mb from the Comeron et al. 2012 map. Run the eMKT through the iMKT R package (installed via devtools::install_github('BGD-UAB/iMKT')) or web service with the 'Compare against whole-genome distribution' option, executing within the dmelProteins Jupyter notebook (python/bash) environment. Run the same eMKT on all 13,753 protein-coding genes to obtain the genome-wide alpha distribution, then compute mean and SD of alpha per set (reference: 0.602, SD 0.032 for high-recombination genes vs 0.44, SD 0.055 genome-wide in the RAL population) and write per-gene results and summary statistics to an output file such as emkt_alpha_results.tsv.

## Related tools

- **iMKT (R package)** (Executes the eMKT with the 'Compare against whole-genome distribution' option on the selected gene set and the genome-wide set) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web-based service alternative for running the four MKT types and comparing selective regimes of a gene set against the genome-wide distribution)
- **Jupyter** (Hosts the dmelProteins python/bash notebook environment in which the iMKTData pipelines and analyses are executed) — github.com/jmurga/iMKTData
- **Python** (Used in the D. melanogaster pipeline notebook for data retrieval and building the DAF/divergence tables consumed by iMKT) — github.com/jmurga/iMKTData
- **bash** (Raw scripts in src/ of iMKTData used to execute pipeline steps for extracting unfolded SFS and divergence by gene) — github.com/BGD-UAB/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Computed mean alpha for the high-recombination set reproduces 0.602 (SD 0.032) in the RAL population
- Computed genome-wide mean alpha reproduces 0.44 (SD 0.055) across all 13,753 protein-coding genes
- High-recombination mean alpha exceeds genome-wide mean alpha, consistent with higher adaptation in high-recombination regions
- Output file (e.g. emkt_alpha_results.tsv) contains per-gene alpha estimates plus mean/SD summary statistics for both gene sets
- The iMKT run reports the expected alpha estimate together with Fisher's exact test P-value and the MKT table (polymorphism/divergence counts for neutral and selected classes)

## Limitations

- Pipelines require downloading large external data (DGN/PopFly FASTA, Comeron 2012 recombination map); paths must be manually changed and some steps need custom parallelization (reference server: 100GB RAM, 16 Xeon CPUs)
- The comparison depends on correctly defined gene subsets; the reference high-recombination threshold (>7 cM/Mb) is specific to the Comeron et al. 2012 map and D. melanogaster RAL population
- MKT-derived methods behave differently according to data properties; iMKT exists to compare standard MKT, aMKT, FWWMKT and eMKT results simultaneously rather than trusting a single estimator

## Evidence

- [abstract] Advanced options allow testing complex hypotheses, illustrated by an application example on whether genes in high recombination regions undergo higher rates of adaptation: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [intro] It allows comparing the selective regimes of a set of coding genes with the genome-wide distribution: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [methods] Variation and divergence data retrieved from PopFly in FASTA format; recombination data from Comeron et al. 2012: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [results] In the RAL population, high-recombination genes show higher alpha (0.602) than the genome-wide set (0.44): [evidence span withheld — anchor status: not_found]
- [readme] iMKT R package computes MK tests on polymorphism and divergence data, optionally auto-downloaded from PopFly: [evidence span withheld — anchor status: external_doc]
