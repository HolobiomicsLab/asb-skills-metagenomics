---
name: allele-abundance-estimation
description: Use when you must quantify how accurately an MLST deconvolution pipeline
  recovers fractional allele composition and complete strain types from simulated
  mixed reads, and need a baseline comparison against abundance-estimation tools like
  Kallisto or strainEST.
when_to_use_negative:
- Input is real WGS tick-sample data rather than controlled simulations — the Kallisto/strainEST
  baseline comparison was validated only on ART-simulated reads with known ground-truth
  allele and strain composition
- The goal is strain deconvolution itself (solving the Allele/Strain Diversity Problem
  via ILP with CPLEX), not benchmarking an already-produced allele/abundance call
  set
- Working with a different organism whose MLST scheme lacks the B. burgdorferi allele
  database and edit-distance structure used to set the d ∈ {5,10,15,20,25} parameter
  grid
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ART
  role: Simulates Illumina-like reads from sampled B. burgdorferi MLST alleles at
    controlled coverage and inter-allele edit distance
- name: Bowtie v0.12.7
  role: Aligns simulated reads to the allele database so mapping statistics can be
    extracted for the B-MLST pipeline
- name: Kallisto
  role: Isoform-abundance baseline run on identical simulated reads to obtain per-locus
    allele calls and abundances for the precision/recall/TVD comparison
- name: strainEST
  role: Reference-based strain-composition baseline compared using soft-precision,
    soft-recall and Earth-Mover's distance across EvoMod simulation scenarios
  repo: https://github.com/compmetagen/strainest
- name: pyemd
  role: Computes Earth-Mover's distance between true and estimated strain distributions
    in the strainEST comparison
  repo: https://github.com/garydoranjr/pyemd
- name: editdistance
  role: Computes Levenshtein distance between alleles to define the d parameter grid
    and allele distances
  repo: https://github.com/aflc/editdistance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:fcee8429817fa4e774e358b7216994d6301c5a7d164b2acb44b60aa02bf48223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/strainest
  tools:
  - ART
  - Bowtie v0.12.7
  - Kallisto
  - Python 2.7
  - editdistance
  - pyemd
  - strainEST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# allele-abundance-estimation

## Summary

Benchmark the allele diversity pipeline (B-MLST) against Kallisto and strainEST on ART-simulated B. burgdorferi MLST read sets, quantifying precision, recall, Total Variation Distance per housekeeping gene, and Earth-Mover's distance for strain-level recovery.

## When to use

Use when you must quantify how accurately an MLST deconvolution pipeline recovers fractional allele composition and complete strain types from simulated mixed reads, and need a baseline comparison against abundance-estimation tools like Kallisto or strainEST.

## When NOT to use

- Input is real WGS tick-sample data rather than controlled simulations — the Kallisto/strainEST baseline comparison was validated only on ART-simulated reads with known ground-truth allele and strain composition
- The goal is strain deconvolution itself (solving the Allele/Strain Diversity Problem via ILP with CPLEX), not benchmarking an already-produced allele/abundance call set
- Working with a different organism whose MLST scheme lacks the B. burgdorferi allele database and edit-distance structure used to set the d ∈ {5,10,15,20,25} parameter grid

## Inputs

- 40 ART-simulated paired-end read sets per locus per parameter combination, generated from the B. burgdorferi MLST allele database
- B. burgdorferi MLST allele database (8 housekeeping genes: clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA; 1726 alleles, 753 strain types)
- Identical simulated reads fed to Kallisto and the B-MLST pipeline
- Full-pipeline simulation datasets for strainEST, including existing-strain (k ∈ {1,2}) and EvoMod1/2/2e/2n/3 novel-strain scenarios

## Outputs

- Comparison CSV of averages ± standard deviations of precision, recall, and TVD per gene versus Kallisto
- Soft-precision, soft-recall and Earth-Mover's distance statistics for the strainEST comparison
- Per-statistic PNG boxplots/summary plots in the results folder

## How to apply

Simulate 40 ART read sets per locus for each parameter combination (coverage c ∈ {30,100,300}, inter-allele edit distance d ∈ {5,10,15,20,25}) from the B. burgdorferi MLST database (8 housekeeping genes, 1726 alleles). Map simulated reads to the allele database with Bowtie v0.12.7 and extract mapping statistics with samtools; run Kallisto via run_kallisto_sim.py with -i 40 on the identical reads to obtain per-locus allele calls and abundances. Run strainEST on the full-pipeline simulation datasets covering both existing-strain benchmarks (k ∈ {1,2}) and novel-strain evolutionary mechanisms (EvoMod1, EvoMod2, EvoMod2e, EvoMod2n, EvoMod3). Compute precision TP/(TP+FP), recall TP/(TP+FN), and Total Variation Distance per gene (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA) for the B-MLST vs Kallisto comparison, plus soft-precision, soft-recall and Earth-Mover's distance (PyEMD) for the strainEST comparison. Average each statistic with standard deviations across parameter combinations, write a comparison CSV, and render per-statistic PNG boxplots in the results folder.

## Related tools

- **ART** (Simulates Illumina-like reads from sampled B. burgdorferi MLST alleles at controlled coverage and inter-allele edit distance)
- **Bowtie v0.12.7** (Aligns simulated reads to the allele database so mapping statistics can be extracted for the B-MLST pipeline)
- **Kallisto** (Isoform-abundance baseline run on identical simulated reads to obtain per-locus allele calls and abundances for the precision/recall/TVD comparison)
- **strainEST** (Reference-based strain-composition baseline compared using soft-precision, soft-recall and Earth-Mover's distance across EvoMod simulation scenarios) — https://github.com/compmetagen/strainest
- **pyemd** (Computes Earth-Mover's distance between true and estimated strain distributions in the strainEST comparison) — https://github.com/garydoranjr/pyemd
- **editdistance** (Computes Levenshtein distance between alleles to define the d parameter grid and allele distances) — https://github.com/aflc/editdistance

## Examples

```
python run_sim.py -i 40 -f simulation_results -c 30 -e 5 -b ../bowtieFolder/ -s ../../samtools -a art_illumina
```

## Evaluation signals

- B-MLST precision exceeds Kallisto at every locus — e.g. clpA 0.99 ± 0.009 vs 0.97 ± 0.014, nifS 0.96 vs 0.89
- Total Variation Distance per gene is comparable between B-MLST and Kallisto while precision is higher
- The proposed method outperforms strainEST across all simulated situations (existing-strain k ∈ {1,2} and EvoMod1/2/2e/2n/3 scenarios)
- Each statistic is reported as an average ± standard deviation over the full grid of coverage and edit-distance parameter combinations (40 iterations each)
- Comparison CSV is produced and per-statistic PNG boxplots render in the results folder without missing-locus gaps

## Limitations

- Comparisons are performed on ART-simulated reads only; real-sample accuracy against Kallisto/strainEST baselines is not established by this benchmark
- Kallisto comparisons are run with 40 iterations (-i 40), and results depend on the chosen coverage c ∈ {30,100,300} and edit-distance d ∈ {5,10,15,20,25} grid; conclusions may not transfer outside this range
- Python 2.7 and old tool versions (Bowtie v0.12.7, samtools v1.3.1, CPLEX 12.6.3.0) are required and may not run on modern systems
- The pipeline's handling of closely related strain clusters relies on ad-hoc post-processing rather than a principled method

## Evidence

- [methods] We compared our results with the results obtained with Kallisto [20], a recent method for isoform abundance estimation: [evidence span withheld — anchor status: not_found]
- [readme] run_kallisto_sim.py with 40 iterations: "`-i` specifies the number of simulations on each gene, default is 40. `-f` specifies the name of the folder to store the results, default is simulation_results. `-c` specifies the coverage to test"
- [methods] ART simulation design: "All reads are simulated using ART [18], following the characteristics of the reads from the real data set"
- [methods] strainEST comparison: "We compared our pipeline with strainEST [13], a recent method to estimate the strain composition and abundance in metagenomics datasets."
- [intro] evaluation on simulated samples: "We evaluate our approach on simulated samples and find that it is accurate in identifying both the fractional allele composition at each housekeeping gene"
