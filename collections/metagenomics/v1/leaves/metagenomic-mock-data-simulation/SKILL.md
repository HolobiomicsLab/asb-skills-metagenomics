---
name: metagenomic-mock-data-simulation
description: Use when a reproducible mock compositional abundance table (with matched
  control/disease phenotype data) is needed to test or benchmark metagenomic feature-enrichment
  or causality-inference methods such as VTwins.
when_to_use_negative:
- You already have a real relative-abundance table (e.g. from curatedMetagenomicData,
  like ZellerG_2014.metaphlan_bugs_list.stool) — use the real-data workflow instead
  of simulating mock data.
- The downstream tool requires raw counts rather than compositional relative abundances
  — uniform random draws plus decostand 'total' normalization produce relative abundances
  only.
- You need biologically realistic covariance/phylogenetic structure in the mock data;
  this simulation draws features independently from a uniform distribution.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3307
tools:
- name: R
  role: Runtime (R 4.0.5) in which the mock data frame and phenotype table are generated
    with set.seed and runif.
  repo: https://github.com/mengqingren/VTwins
- name: vegan
  role: Provides decostand with method='total' on margin 1 to convert the raw uniform
    draws into per-sample relative abundances.
- name: tidyverse
  role: tidyverse_1.3.1 dependency loaded alongside VTwins for the mock-data example
    workflow.
- name: VTwins
  role: 'Downstream consumer of the mock dataset: pair_find uses the simulated abundance
    and phenotype tables to demonstrate phenotype-enriched feature identification.'
  repo: https://github.com/mengqingren/VTwins
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.scib.2023.10.024
    title: 'VTwins: inferring causative microbial features from metagenomic data of
      limited samples'
  source_content_hash: sha256:862b78c109f2388c2e5f59b22522739c0eaa4649006f67740bd1d45586f138e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mengqingren/VTwins
  tools:
  - R
  - VTwins
  - tidyverse
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metagenomic-mock-data-simulation

## Summary

Generates a reproducible mock compositional metagenomic abundance dataset (120 samples × 10 features from runif with set.seed(12345), normalized to relative abundances with vegan::decostand 'total') plus a grp1/grp2 phenotype table, as used by the VTwins paper to test sensitivity of causative-feature inference on limited samples.

## When to use

Use when a reproducible mock compositional abundance table (with matched control/disease phenotype data) is needed to test or benchmark metagenomic feature-enrichment or causality-inference methods such as VTwins.

## When NOT to use

- You already have a real relative-abundance table (e.g. from curatedMetagenomicData, like ZellerG_2014.metaphlan_bugs_list.stool) — use the real-data workflow instead of simulating mock data.
- The downstream tool requires raw counts rather than compositional relative abundances — uniform random draws plus decostand 'total' normalization produce relative abundances only.
- You need biologically realistic covariance/phylogenetic structure in the mock data; this simulation draws features independently from a uniform distribution.

## Inputs

- R random seed (set.seed(12345)) controlling reproducibility of the simulation
- Simulation parameters: 1200 draws from runif(min = 1e-5, max = 1), nrow = 120, ncol = 10
- Phenotype design: 60 grp1 (control) and 60 grp2 (disease) samples, ids Sample1..Sample120

## Outputs

- Mock relative abundance data frame (120 samples × 10 features, rows Sample1..Sample120, columns Feature1..Feature10) after decostand 'total' normalization
- Phenotype data frame with columns id and grp (grp1/grp2), sample order identical to the abundance table
- Optionally a tab-separated file (e.g. test.data.txt) of the normalized table for reuse in tests

## How to apply

In R, seed the RNG with set.seed(12345) and generate a 120-row × 10-column data frame via matrix(runif(1200, min = 1e-5, max = 1), nrow = 120, ncol = 10), naming rows Sample1..Sample120 and columns Feature1..Feature10. Because raw uniform draws are not compositional, normalize each sample to relative abundance with vegan::decostand(dataset, method = 'total', 1) on margin 1. Build a two-column phenotype data frame with colnames id and grp, assigning the first 60 samples (Sample1..Sample60) to grp1 (control) and the last 60 (Sample61..Sample120) to grp2 (disease). Critically, keep the same sample order in the abundance data frame and phenotype data frame, with samples clustered and placed according to grp1 and grp2 — the VTwins pair-finding logic depends on this ordering. The mock dataset (and a real dataset from the VTwins.Linux repo) can be used directly as input to VTwins::pair_find to exercise the enrichment workflow and reproduce the reported sensitivity advantage.

## Related tools

- **R** (Runtime (R 4.0.5) in which the mock data frame and phenotype table are generated with set.seed and runif.) — https://github.com/mengqingren/VTwins
- **vegan** (Provides decostand with method='total' on margin 1 to convert the raw uniform draws into per-sample relative abundances.)
- **tidyverse** (tidyverse_1.3.1 dependency loaded alongside VTwins for the mock-data example workflow.)
- **VTwins** (Downstream consumer of the mock dataset: pair_find uses the simulated abundance and phenotype tables to demonstrate phenotype-enriched feature identification.) — https://github.com/mengqingren/VTwins

## Examples

```
set.seed(12345); dataset <- data.frame(matrix(runif(1200, min=1e-5, max=1), nrow=120, ncol=10)); dataset.normalized <- decostand(dataset, method='total', 1)
```

## Evaluation signals

- Resulting abundance data frame has exactly 120 rows named Sample1..Sample120 and 10 columns named Feature1..Feature10.
- After decostand(method='total', margin=1), each sample's feature values sum to 1 (relative abundance property).
- Phenotype data frame has exactly two columns (id, grp) with 60 grp1 entries followed by 60 grp2 entries in the same sample order as the abundance table.
- Because set.seed(12345) precedes runif, regenerated mock data are numerically identical across runs, enabling exact reproduction of pair_find outputs.
- Feeding the mock data to pair_find at PvalueCutoff=0.01 yields a Results.csv with the expected columns (Species, Enriched, Decre.aveRank.P, Incre.aveRank.P, FDR columns, Ctlmean, Dismean, log2FC).

## Limitations

- If the number of sample pairs produced downstream is less than 10, pair_find returns nothing — the 120-sample design above is sized to avoid this.
- Sample order must be identical between the relative abundance data frame and phenotype data frame, with samples clustered and placed according to grp1 and grp2; violating this breaks pairing.
- Dependencies are pinned to specific versions in the paper/README (R version 4.0.5, tidyverse_1.3.1); other versions may not be validated.
- The mock data are independent uniform draws, not realistic compositional microbiome data; findings on it validate mechanics, not biological plausibility.

## Evidence

- [intro] The goal of `VTwins` is to perform phenotype-enriched features including species or pathways for metagenomic sequencing or 16S sequencing.: "The goal of `VTwins` is to perform phenotype-enriched features including species or pathways for metagenomic sequencing or 16S sequencing."
- [readme] Mock data generation code: "set.seed(12345)
dataset <- data.frame(matrix(runif(1200, min = 1e-5, max = 1),nrow = 120,ncol = 10))"
- [methods] decostand normalization step: "dataset.normalized <- decostand(dataset,method = "total",1) #normalization for fetures like species's relative abundance"
- [readme] Sample ordering requirement: "Must keep the `Same Order` of samples in  relative abundance dataframe and phenotype dataframe. And we also keep the samples `clustered and placed` according the `grp1 and grp2`"
