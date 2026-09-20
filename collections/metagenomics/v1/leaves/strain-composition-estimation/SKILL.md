---
name: strain-composition-estimation
description: Use when WGS reads from a bacterial sample (e.g. B. burgdorferi ticks)
  must be decomposed into constituent MLST strain types and allele/strain proportions
  despite within-host heterogeneity.
when_to_use_negative:
- The reference strain/allele database for the organism is unavailable — de novo approaches
  such as DESMAN are the alternative the paper cites
- The goal is SNP-level variant calling on a single clonal isolate rather than deconvolution
  of a multi-strain mixture
- The sample's loci do not map to a characterized MLST scheme, since the pipeline
  requires known alleles and strain types
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: CPLEX 12.6.3.0
  role: Solves the ADP 0-1 ILP and the Strain Diversity Problem MILP via its Python
    API
- name: Bowtie v0.12.7
  role: Maps WGS reads onto the MLST allele database before allele calling
- name: samtools v1.3.1
  role: Extracts mapping statistics from SAM files for the ILP input text files
- name: ART
  role: Simulates reads from MLST alleles to generate the ADP/SDP benchmark datasets
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- name: Kallisto
  role: Baseline method for per-locus allele calls and abundance comparison (run_kallisto_sim.py
    with -i 40)
- name: strainEST
  role: Baseline strain-composition method compared against via soft-precision, soft-recall
    and Earth-Mover's distance
  repo: https://github.com/compmetagen/strainest
- name: GAML
  role: Likelihood-based selection among multiple optimal ILP solutions (post-processing)
- name: pyemd
  role: Computes Earth-Mover's distance for the strainEST comparison
  repo: https://github.com/garydoranjr/pyemd
- name: editdistance
  role: Computes Levenshtein distances between alleles used in the SDP objective
  repo: https://github.com/aflc/editdistance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:b6a322bfa0f630962cc3c68e3da4acff29ed3e26ce27642380ce530badbf0b04
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - Kallisto
  - Python 2.7
  - editdistance
  - pyemd
  - samtools v1.3.1
  - strainEST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# strain-composition-estimation

## Summary

Infers the set of MLST strain types and their relative abundances present in a within-host heterogeneous sample by first solving the Allele Diversity Problem (allele composition per housekeeping locus) and then the Strain Diversity Problem, both formulated and solved as mixed integer linear programs with CPLEX.

## When to use

Use when WGS reads from a bacterial sample (e.g. B. burgdorferi ticks) must be decomposed into constituent MLST strain types and allele/strain proportions despite within-host heterogeneity.

## When NOT to use

- The reference strain/allele database for the organism is unavailable — de novo approaches such as DESMAN are the alternative the paper cites
- The goal is SNP-level variant calling on a single clonal isolate rather than deconvolution of a multi-strain mixture
- The sample's loci do not map to a characterized MLST scheme, since the pipeline requires known alleles and strain types

## Inputs

- Paired-end FASTQ reads from B. burgdorferi samples (downloaded via fastq-dump from SRA accession lists such as SRR2034336)
- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 alleles, 753 strain types)
- Edit-distance matrices for each MLST gene (editDist folder)
- ART-simulated read sets parameterized by coverage c∈{30,100,300} and inter-allele edit distance d∈{5,10,15,20,25}

## Outputs

- geneX_proportions.csv files listing identified alleles and their proportions per locus per sample
- sampleX_strainsAndProportions.csv files listing inferred strain types and their abundances per sample
- Comparison CSV of averages ± standard deviations (precision, recall, TVD, soft-precision/soft-recall, EMD) and PNG summary plots against Kallisto and strainEST

## How to apply

Map reads with Bowtie v0.12.7 onto the reference MLST allele database and extract per-locus coverage information from SAM files with samtools; solve the 0-1 ILP for the Allele Diversity Problem per locus, post-processing multiple optimal solutions with the GAML likelihood model, then compute allele proportions. With per-sample allele calls in hand, solve the Strain Diversity Problem MILP (CPLEX 12.6.3.0 Python API) that minimizes the number of novel strains while staying close to observed strain types and respecting allele proportions, using edit-distance matrices for each gene (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA). Control the global solver run with a relative gap (-g, default 5%) and time limit (-timelim, default 600 s); the program stops only when both are satisfied. Validate by benchmarking on ART-simulated reads and comparing precision/recall against Kallisto (allele level) and strainEST (strain level, using soft-precision/soft-recall and Earth-Mover's distance).

## Related tools

- **CPLEX 12.6.3.0** (Solves the ADP 0-1 ILP and the Strain Diversity Problem MILP via its Python API)
- **Bowtie v0.12.7** (Maps WGS reads onto the MLST allele database before allele calling)
- **samtools v1.3.1** (Extracts mapping statistics from SAM files for the ILP input text files)
- **ART** (Simulates reads from MLST alleles to generate the ADP/SDP benchmark datasets) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- **Kallisto** (Baseline method for per-locus allele calls and abundance comparison (run_kallisto_sim.py with -i 40))
- **strainEST** (Baseline strain-composition method compared against via soft-precision, soft-recall and Earth-Mover's distance) — https://github.com/compmetagen/strainest
- **GAML** (Likelihood-based selection among multiple optimal ILP solutions (post-processing))
- **pyemd** (Computes Earth-Mover's distance for the strainEST comparison) — https://github.com/garydoranjr/pyemd
- **editdistance** (Computes Levenshtein distances between alleles used in the SDP objective) — https://github.com/aflc/editdistance

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -g 5 -timelim 600 -pathToDistMat editDist
```

## Evaluation signals

- ADP precision should exceed Kallisto at every locus (e.g. clpA 0.99 ± 0.009 vs 0.97 ± 0.014; nifS 0.96 vs 0.89) with comparable Total Variation Distance
- Strain-level soft-precision/soft-recall and EMD should outperform strainEST across all simulated situations, including k∈{1,2} existing-strain benchmarks and EvoMod1/EvoMod2/EvoMod2e/EvoMod2n/EvoMod3 novel-strain scenarios
- Each sample folder in variantsAndProp must contain exactly 8 geneX_proportions.csv files, one per MLST locus
- Global solver must terminate only when both the relative gap (default 5%) and the time limit (default 600 s) conditions are met
- Number of inferred novel strains should stay small — on the 24 real tick samples the pipeline inferred 60 novel and 10 existing strains

## Limitations

- Clusters of closely related strains are handled by ad-hoc post-processing rather than a principled method; more principled approaches remain unexplored
- Both the Allele Diversity Problem and the Strain Diversity Problem are NP-hard, so the MILP relies on gap/time limits and may stop at suboptimal solutions
- If the 0-1 ILP has multiple optimal solutions, an arbitrary one may be selected unless GAML post-processing is applied
- Supplementary methods, figures and tables (Additional file 1) are referenced but not fully included in the article text

## Evidence

- [abstract] First we process each sample individually by assigning it, for each locus in the MLST scheme, a set of alleles and a proportion for each allele.: "First we process each sample individually by assigning it, for each locus in the MLST scheme, a set of alleles and a proportion for each allele."
- [abstract] We solve both problems using mixed integer linear programming (MILP).: "We solve both problems using mixed integer linear programming (MILP)."
- [methods] Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [methods] For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls.: "For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls."
- [readme] The -oc argument specifies different formulation of the objective function, 'all' includes all components of the objective function, 'noPropAndErr' omits the proportion and error component.: "The -oc argument specifies different formulation of the objective function, 'all' includes all components of the objective function, 'noPropAndErr' omits the proportion and error component."
