---
name: isoform-abundance-quantification-comparison
description: Use when you have simulated read sets mapped to a known allele database
  and need to validate fractional allele composition calls against an established
  isoform/abundance quantification baseline such as Kallisto.
when_to_use_negative:
- Input is real metagenomic tick sample data rather than simulated reads with known
  ground-truth allele composition — the precision/recall/TVD comparison requires true
  allele calls
- The organism or typing scheme is not a multi-locus sequence typing framework with
  a curated reference allele database (e.g. the B. burgdorferi MLST scheme)
- The goal is strain-type level comparison including novel strains (EvoMod1–EvoMod3
  mechanisms), which requires the strainEST comparison with soft-precision/soft-recall/EMD
  rather than Kallisto alone
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Kallisto
  role: Baseline isoform abundance estimator run on identical simulated reads to obtain
    per-locus allele calls and abundances for comparison
- name: ART
  role: Simulates Illumina reads from MLST alleles at controlled coverage and inter-allele
    edit distance
  repo: https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- name: Bowtie v0.12.7
  role: Aligns simulated reads to the alleles of the database
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools v1.3.1
  role: Extracts mapping statistics from sam files after mapping
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: strainEST
  role: Secondary baseline for strain-level composition comparison using soft-precision,
    soft-recall and Earth-Mover's distance
  repo: https://github.com/compmetagen/strainest
- name: editdistance (aflc)
  role: Computes Levenshtein edit distance between alleles
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Earth Mover's Distance calculations for the strainEST comparison
  repo: https://github.com/garydoranjr/pyemd
- name: CPLEX 12.6.3.0
  role: Solves the Allele Diversity Problem ILP whose abundance estimates are being
    benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:75acc168ba8235de6bb590e48b130532f383d320c02769844d8511d535911654
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - Kallisto
  - PyEMD
  - Python 2.7
  - editdistance (aflc)
  - samtools v1.3.1
  - strainEST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# isoform-abundance-quantification-comparison

## Summary

Benchmark an MLST deconvolution pipeline's per-locus allele abundance estimates against Kallisto, a likelihood-based isoform abundance estimator, using ART-simulated reads mapped with Bowtie v0.12.7 to the B. burgdorferi MLST allele database.

## When to use

Use when you have simulated read sets mapped to a known allele database and need to validate fractional allele composition calls against an established isoform/abundance quantification baseline such as Kallisto.

## When NOT to use

- Input is real metagenomic tick sample data rather than simulated reads with known ground-truth allele composition — the precision/recall/TVD comparison requires true allele calls
- The organism or typing scheme is not a multi-locus sequence typing framework with a curated reference allele database (e.g. the B. burgdorferi MLST scheme)
- The goal is strain-type level comparison including novel strains (EvoMod1–EvoMod3 mechanisms), which requires the strainEST comparison with soft-precision/soft-recall/EMD rather than Kallisto alone

## Inputs

- ART-simulated FASTQ read sets (40 iterations per gene per parameter combination)
- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 alleles, 753 strain types)
- SAM/BAM mapping statistics from Bowtie v0.12.7 processed with samtools v1.3.1
- Kallisto allele call and abundance outputs per simulated sample

## Outputs

- Comparison CSV of averaged precision, recall, and Total Variation Distance ± standard deviations per gene vs Kallisto
- Per-statistic PNG boxplots/summary plots in the results folder
- Per-locus allele calls and estimated allele proportions from Kallisto (40 iterations)

## How to apply

Generate 40 ART-simulated read sets per locus per parameter combination (coverage c ∈ {30,100,300}, inter-allele edit distance d ∈ {5,10,15,20,25}) from the B. burgdorferi MLST allele database (8 housekeeping genes, 1726 alleles). Map reads with Bowtie v0.12.7 and extract mapping statistics with samtools v1.3.1, then run Kallisto via run_kallisto_sim.py with 40 iterations (-i 40) on the identical simulated reads to obtain per-locus allele calls and abundances. Compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and Total Variation Distance per gene (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA), average them with standard deviations across parameter combinations, and write the results to a comparison CSV with per-statistic PNG summary plots. Use the aflc/editdistance package for edit distances and garydoranjr/pyemd for Earth-Mover's distances when additionally benchmarking against strainEST (soft-precision, soft-recall, EMD). Rationale: Kallisto is a recent likelihood-based method for isoform abundance estimation, making it a directly comparable baseline for fractional allele composition accuracy.

## Related tools

- **Kallisto** (Baseline isoform abundance estimator run on identical simulated reads to obtain per-locus allele calls and abundances for comparison)
- **ART** (Simulates Illumina reads from MLST alleles at controlled coverage and inter-allele edit distance) — https://www.niehs.nih.gov/research/resources/software/biostatistics/art/
- **Bowtie v0.12.7** (Aligns simulated reads to the alleles of the database) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools v1.3.1** (Extracts mapping statistics from sam files after mapping) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **strainEST** (Secondary baseline for strain-level composition comparison using soft-precision, soft-recall and Earth-Mover's distance) — https://github.com/compmetagen/strainest
- **editdistance (aflc)** (Computes Levenshtein edit distance between alleles) — https://github.com/aflc/editdistance
- **PyEMD** (Earth Mover's Distance calculations for the strainEST comparison) — https://github.com/garydoranjr/pyemd
- **CPLEX 12.6.3.0** (Solves the Allele Diversity Problem ILP whose abundance estimates are being benchmarked)

## Examples

```
python run_kallisto_sim.py -i 40 -f kallisto_results -c 30 -e 10
```

## Evaluation signals

- Proposed pipeline precision exceeds Kallisto at every locus (e.g. clpA 0.99 ± 0.009 vs 0.97 ± 0.014, nifS 0.96 vs 0.89)
- Total Variation Distance per gene is comparable between B-MLST and Kallisto across parameter combinations
- Averages ± standard deviations are reported across all parameter combinations (coverage 30/100/300 × edit distance 5–25) and written to the comparison CSV
- Identical simulated read sets are used for both the pipeline and Kallisto (-i 40 iterations), so differences reflect the estimators not the data
- Per-statistic PNG summary plots are rendered in the results folder and are consistent with the CSV averages

## Limitations

- Python 2.7 and the CPLEX Python API (academic license) are required; PyEMD explicitly does not support Python 3
- Kallisto defaults to 40 iterations (-i 40) in the simulation harness; other iteration counts are not validated in this workflow
- Coverage and edit distance grids are fixed to c ∈ {30,100,300} and d ∈ {5,10,15,20,25}; parameter combinations outside this grid are untested
- Simulation relies on ART reads mimicking characteristics of the real SRR2034336 sample; other read profiles are not characterized

## Evidence

- [methods] Kallisto baseline rationale: "we compared our results with the results obtained with Kallisto [20], a recent method for isoform abundance estimation"
- [methods] Bowtie mapping of simulated reads: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [readme] Edit distance package: "In computing edit distance, we used the package by https://github.com/aflc/editdistance and we hereby acknowledge Hiroyuki Tanaka"
- [readme] PyEMD for EMD: "We also use the package PyEMD for EMD calculations, where we hereby acknowledge Gary Doran for his PyEMD package"
- [readme] Simulation harness parameters: "`-i` specifies the number of simulations on each gene, default is 40. `-f` specifies the name of the folder to store the results, default is simulation_results. `-c` specifies the coverage to test"
