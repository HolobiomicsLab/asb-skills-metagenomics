---
name: mlst-allele-proportion-normalization
description: Use when the ADP ILP has already identified the allele set for an MLST
  locus and you need fractional allele abundances per gene from the per-sample, per-gene
  read mapping files.
when_to_use_negative:
- Allele selection (the ILP itself) has not yet been run — proportions cannot be computed
  before the allele set is fixed
- Input is not short-read WGS mapped to a known MLST allele database (e.g. de novo
  assemblies without read-to-allele mapping scores)
- The goal is strain-level deconvolution rather than per-locus allele proportions
  — that requires the separate globalStrainDiversity.py MILP stage
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: CPLEX 12.6.3.0
  role: Python API used to solve the 0-1 ILP whose selected alleles are the basis
    for the proportion computation
  repo: https://github.com/WGS-TB/MLST
- name: GAML
  role: Likelihood-based post-processing used to pick one solution when the ILP has
    multiple optima, prior to proportion computation
- name: Bowtie v0.12.7
  role: Maps sample reads onto the known allele database, producing the .sam files
    from which the per-gene mapping scores are extracted
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:f7e28c228b5e317b126addc191f5b43801dc35a2f04685d2b911d76eb32ec227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - Python 2.7
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mlst-allele-proportion-normalization

## Summary

Converts the 0-1 ILP allele-selection output of the Allele Diversity Problem (ADP) into per-locus allele proportions by weighting each retained read across the alleles attaining its optimal mapping score and normalizing to a distribution. This supplies the fractional allele abundances per MLST housekeeping gene that feed the downstream Strain Diversity Problem.

## When to use

Use when the ADP ILP has already identified the allele set for an MLST locus and you need fractional allele abundances per gene from the per-sample, per-gene read mapping files.

## When NOT to use

- Allele selection (the ILP itself) has not yet been run — proportions cannot be computed before the allele set is fixed
- Input is not short-read WGS mapped to a known MLST allele database (e.g. de novo assemblies without read-to-allele mapping scores)
- The goal is strain-level deconvolution rather than per-locus allele proportions — that requires the separate globalStrainDiversity.py MILP stage

## Inputs

- Selected allele set per locus from the ADP 0-1 ILP solution (post-GAML tie-breaking)
- Per-sample, per-gene read mapping files (sampleX_geneY_paired_reads.txt in the variantsAndProp folder)
- Weighted bipartite read-allele incidence matrix M with normalized Phred mismatch scores (non-mapping reads set to infinity)

## Outputs

- geneX_proportions.csv (8 files per sample, one per MLST locus) containing alleles and their normalized proportions in the sample's variantsAndProp folder
- Allele proportion vector per gene summing to 1, consumed by the Strain Diversity Problem MILP

## How to apply

After solving the 0-1 ILP with CPLEX (Python 2.7 API) and resolving ties with GAML, compute proportions for each of the 8 B. burgdorferi MLST housekeeping genes. For each gene, first discard reads that map equally well (identical minimal weighted Phred mismatch score, with scores normalized over the 33-126 range) onto all selected alleles, since they carry no discriminating information. For every retained read, add weight 1/h to each of the h selected alleles that attain the chosen optimal mapping score, then normalize the accumulated allele weights to sum to 1. Write the resulting alleles and their normalized proportions to geneX_proportions.csv in the sample's variantsAndProp folder; these proportions are later consumed by the global Strain Diversity MILP.

## Related tools

- **CPLEX 12.6.3.0** (Python API used to solve the 0-1 ILP whose selected alleles are the basis for the proportion computation) — https://github.com/WGS-TB/MLST
- **GAML** (Likelihood-based post-processing used to pick one solution when the ILP has multiple optima, prior to proportion computation)
- **Bowtie v0.12.7** (Maps sample reads onto the known allele database, producing the .sam files from which the per-gene mapping scores are extracted) — https://github.com/WGS-TB/MLST

## Examples

```
python alleleDiversity.py -s SRR2034336
```

## Evaluation signals

- The script produces exactly 8 geneX_proportions.csv files (one per locus) in each sample's variantsAndProp folder
- Each per-gene proportion vector sums to 1 after normalization
- Reads mapping equally well onto all selected alleles are excluded from the weight computation (verifiable by inspecting the intermediate output printed to screen)
- Resource usage stays feasible: reference workload SRR2034336 averages ~20,112 ILP variables per gene with ~1.5GB peak RAM, ~33 min for all 8 genes on a 4-CPU machine
- Simulated-data accuracy matches reported levels (e.g. average B-MLST precision 0.98 +/- 0.011)

## Limitations

- Proportions depend on the ILP's parsimonious allele selection; when multiple optimal solutions exist, GAML tie-breaking determines the final allele set and hence the proportions
- Reads mapping equally well to all selected alleles are discarded, so coverage effectively contributing to proportion estimation may be lower than total mapped coverage
- Handling of closely related strain clusters is acknowledged as ad-hoc post-processing rather than a principled method

## Evidence

- [other] Compute allele proportions: discard reads mapping equally well onto all selected alleles, add weight 1/h to each of the h alleles attaining the chosen optimal score for each retained read, then normalize the allele weights to sum to 1.: [evidence span withheld — anchor status: not_found]
- [methods] once the alleles have been identified for a given locus, we compute the proportion of each allele.: "once the alleles have been identified for a given locus, we compute the proportion of each allele."
- [readme] This script will output some intermediate results to the screen and create 8 CSV files (for each locus) in each of the sample folder in `variantsAndProp`, named as `geneX_proportions.csv` which contains the alleles and their proportions identified at `geneX`.: "create 8 CSV files (for each locus) in each of the sample folder in `variantsAndProp`, named as `geneX_proportions.csv` which contains the alleles and their proportions"
- [methods] Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
