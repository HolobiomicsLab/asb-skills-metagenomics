---
name: allele-call-accuracy-evaluation-tvd
description: Use when validating an allele-diversity or strain-deconvolution stage
  against known simulated allele mixtures, and you need per-locus precision/recall
  and TVD statistics to compare with published baselines.
when_to_use_negative:
- Input is real tick-sample WGS data with unknown allele composition — no ground truth
  exists to compute Precision/Recall/TVD against
- Evaluating the second Strain Diversity Problem (SDP) stage, which is validated with
  random strain-type distributions and perfect allele calls rather than read-level
  ADP simulation
- The goal is comparing against alternative abundance estimators like Kallisto or
  strainEST using their native outputs rather than the ADP simulation framework
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: run_sim.py
  role: Orchestrates allele-panel simulation and read simulation for the ADP benchmark
    across coverage and edit-distance parameters
  repo: https://github.com/WGS-TB/MLST
- name: ART
  role: Simulates Illumina reads following the characteristics of the real tick-sample
    reads
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- name: Bowtie v0.12.7
  role: Aligns simulated reads to the alleles of the MLST database
- name: samtools v1.3.1
  role: Extracts mapping information from sam files for proportion estimation
- name: CPLEX 12.6.3.0
  role: Solves the ADP 0-1 ILP via its Python API to recover the simulated allele
    set
- name: GAML
  role: Likelihood-based selection among multiple optimal ILP solutions
- name: editdistance
  role: Computes Levenshtein distance used to pick allele panels within edit distance
    d and to evaluate TVD via EMD
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Earth Mover's Distance package used for TVD/EMD calculation between predicted
    and true allele distributions
  repo: https://github.com/garydoranjr/pyemd
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:e84705330e38864a461451b062a54ff39110738814378cbbdc7b11eef4ee66ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance
  - run_sim.py
  - samtools v1.3.1
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# allele-call-accuracy-evaluation-tvd

## Summary

Benchmarking protocol that quantifies how accurately the Allele Diversity Problem (ADP) stage of the B. burgdorferi MLST pipeline recovers simulated fractional allele compositions, using Precision, Recall and Total Variation Distance (TVD) aggregated per locus.

## When to use

Use when validating an allele-diversity or strain-deconvolution stage against known simulated allele mixtures, and you need per-locus precision/recall and TVD statistics to compare with published baselines.

## When NOT to use

- Input is real tick-sample WGS data with unknown allele composition — no ground truth exists to compute Precision/Recall/TVD against
- Evaluating the second Strain Diversity Problem (SDP) stage, which is validated with random strain-type distributions and perfect allele calls rather than read-level ADP simulation
- The goal is comparing against alternative abundance estimators like Kallisto or strainEST using their native outputs rather than the ADP simulation framework

## Inputs

- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 known alleles)
- Simulated allele panels per locus with known allele identities and true proportions
- ART-simulated WGS reads mimicking real tick-sample characteristics (coverage c in {30,100,300})
- SAM/BAM mapping files from Bowtie v0.12.7 processed with samtools v1.3.1

## Outputs

- adp_simulation_results.csv with per-locus averages and standard deviations of Precision, Recall and TVD across all (c,d) parameter combinations
- Predicted allele sets and normalized allele proportions per simulated locus

## How to apply

For each of the 8 B. burgdorferi MLST housekeeping loci (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA), draw a random allele count k in [2,7], pick one database allele plus k-1 others within edit distance d, and assign random proportions summing to 1, running 40 simulations per (c,d) combination with coverage c in {30,100,300} and edit distance d in {5,10,15,20,25}. Simulate reads with ART following the real tick-sample characteristics, map with Bowtie v0.12.7, extract mapping info with samtools, and solve the ADP as a 0-1 ILP with the CPLEX Python API (selecting among multiple optima with GAML). Compute allele proportions by weighting each selected allele from the ILP-selected read mappings and normalizing after filtering reads mapping equally well to all selected alleles, then compute Precision, Recall and TVD against truth and aggregate per-locus averages and standard deviations into adp_simulation_results.csv. A correct reproduction matches Table 1: average precision ~0.98 ± 0.011, average recall ~0.96 ± 0.026, and per-gene TVD in the ~0.07-0.12 range.

## Related tools

- **run_sim.py** (Orchestrates allele-panel simulation and read simulation for the ADP benchmark across coverage and edit-distance parameters) — https://github.com/WGS-TB/MLST
- **ART** (Simulates Illumina reads following the characteristics of the real tick-sample reads) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- **Bowtie v0.12.7** (Aligns simulated reads to the alleles of the MLST database)
- **samtools v1.3.1** (Extracts mapping information from sam files for proportion estimation)
- **CPLEX 12.6.3.0** (Solves the ADP 0-1 ILP via its Python API to recover the simulated allele set)
- **GAML** (Likelihood-based selection among multiple optimal ILP solutions)
- **editdistance** (Computes Levenshtein distance used to pick allele panels within edit distance d and to evaluate TVD via EMD) — https://github.com/aflc/editdistance
- **PyEMD** (Earth Mover's Distance package used for TVD/EMD calculation between predicted and true allele distributions) — https://github.com/garydoranjr/pyemd

## Examples

```
python run_sim.py -i 40 -f simulation_results -c 30 -e 15 -b ../bowtieFolder/ -s ../../samtools -a art_illumina
```

## Evaluation signals

- Average precision across loci should reach ~0.98 ± 0.011, matching Table 1
- Average recall across loci should reach ~0.96 ± 0.026, matching Table 1
- Per-gene TVD should fall in the ~0.07-0.12 range, matching Table 1
- Predicted per-locus allele proportions must sum to 1 after normalizing and filtering reads mapping equally well to all selected alleles
- The output adp_simulation_results.csv must contain per-locus averages and standard deviations aggregated across all 12 (c,d) combinations × 40 simulations

## Limitations

- The evaluation only covers simulated reads with k∈[2,7] alleles and edit distances d∈{5,10,15,20,25}; real samples may have higher or more structured heterogeneity
- When the 0-1 ILP has multiple optimal solutions, GAML-based selection introduces a stochastic component in the recovered proportions
- Accuracy metrics are defined against perfect knowledge of the truth distribution and do not capture novel-allele detection failures outside the database

## Evidence

- [methods] ADP simulation design: "For each locus of the Borrelia MLST scheme, we drew a random number k∈[2,7], selected a random allele from the database"
- [methods] GAML post-processing of multiple optima: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML"
- [methods] ART simulation parameters: "All reads are simulated using ART [18], following the characteristics of the reads from the real data set"
- [readme] run_sim.py usage and arguments: "python run_sim.py [-h] [-i NUMOFITER] [-f SIMULATIONRESULTFOLDER] [-c COVERAGE] [-e EDITDIST] [-fp FULLPAPER] [-b BOWTIE] [-s SAMTOOLS] [-a ART]"
- [readme] Simulation iteration and coverage defaults: "`-i` specifies the number of simulations on each gene, default is 40. `-f` specifies the name of the folder to store the results, default is simulation_results."
- [methods] Mapping and extraction tooling: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
