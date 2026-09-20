---
name: assembly-contiguity-assessment
description: Use when planning or evaluating de novo diploid genome assembly from
  10x Chromium Linked-Read data, especially to choose fragment length (Mu_F) or physical
  coverage (CF) parameters and predict contig N50 outcomes at a fixed total coverage
  such as C = 56x.
when_to_use_negative:
- Input data are long-read (PacBio/ONT) or non-barcode-based short reads rather than
  10x Chromium Linked-Reads — the LRTK-SIM model and CF/CR parameter space are specific
  to partitioned linked-read data
- The goal is variant calling or phase-block extension rather than contig contiguity
  — long fragments extend phase blocks but decrease contig N50, so the two objectives
  pull in opposite directions
- No high-quality reference exists and only N50 is available as a metric — N50 alone
  is misleading for accuracy; accuracy assessment (NA50) requires a reference-backed
  gold standard
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: LRTK-SIM
  role: Simulates 10x Linked-Read data following the Chromium workflow, sweeping CF,
    CR, Mu_F and other config parameters to generate realistic test libraries at constant
    total coverage
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: gen_fasta.py
  role: Auxiliary LRTK-SIM program that generates two haploid fasta source genomes
    by inserting SNVs from a VCF into the reference sequence
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Supernova2
  role: De novo assembler used to build diploid assemblies from each simulated or
    real 10x library, from which contig/scaffold contiguity metrics are extracted
- name: Evaluate_diploid_assembly
  role: Jupyter-based code for evaluating and comparing diploid assemblies produced
    under different parameters
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:a8cbde8c42680f9fcbdcdad3ce2777d9c715e79c8d803d8213cc7292e6d050fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - Evaluate_diploid_assembly
  - LRTK-SIM
  - Python
  - Supernova2
  - gen_fasta.py
  - multiprocessing
  - numpy
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# assembly-contiguity-assessment

## Summary

Simulate 10x Linked-Read libraries with LRTK-SIM across controlled parameter combinations (C, CF, CR, Mu_F), assemble them de novo with Supernova2, and tabulate contig/scaffold contiguity metrics (e.g. N50, NA50) to determine how fragment length and physical coverage affect diploid assembly quality.

## When to use

Use when planning or evaluating de novo diploid genome assembly from 10x Chromium Linked-Read data, especially to choose fragment length (Mu_F) or physical coverage (CF) parameters and predict contig N50 outcomes at a fixed total coverage such as C = 56x.

## When NOT to use

- Input data are long-read (PacBio/ONT) or non-barcode-based short reads rather than 10x Chromium Linked-Reads — the LRTK-SIM model and CF/CR parameter space are specific to partitioned linked-read data
- The goal is variant calling or phase-block extension rather than contig contiguity — long fragments extend phase blocks but decrease contig N50, so the two objectives pull in opposite directions
- No high-quality reference exists and only N50 is available as a metric — N50 alone is misleading for accuracy; accuracy assessment (NA50) requires a reference-backed gold standard

## Inputs

- Reference genome fasta (e.g. hg38) plus a VCF of variants for diploid source-genome construction
- LRTK-SIM configuration files specifying Path_Fastahap1/Path_Fastahap2, processors, CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, and Hap=2
- Simulated paired-end gzipped FASTQ files per library (X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz)

## Outputs

- Supernova2 de novo diploid assemblies (including megabubbles output) for each simulated or real library
- Table of contiguity metrics (contig N50, scaffold N50/NA50) per library versus Mu_F, W_mu_FL, and CF
- Validated parameter-space conclusions: optimal CF window (~332x-823x, worsening >1,000x) and optimal W_mu_FL (~50-150 kb) at constant C

## How to apply

Generate two haploid reference fasta files from a VCF and the hg38 reference using gen_fasta.py (SNVs only, Hap=2), then write LRTK-SIM configuration files setting Path_Fastahap1/Path_Fastahap2, CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, and processors so that total coverage C is held constant while Mu_F (fragment length, in Kb) and CF are varied. Run 'python LRTK-SIM.py <config folder>' to simulate multiple libraries in parallel, each producing gzipped paired-end fastqs (X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz), then assemble each library de novo with Supernova2 (requesting megabubbles output for diploid comparison). Extract contig-length metrics (contig N50; NA50 when a high-quality reference exists for accuracy comparison) and tabulate them against Mu_F / length-weighted fragment length (W_mu_FL) and CF. Judge success by the expected trends: contig contiguity decreases as fragment length and physical coverage increase; the optimal CF window is ~332x-823x with quality worsening above 1,000x at C = 56x, and the optimal W_mu_FL is ~50-150 kb; with broadly optimal parameters ~80% of the genome assembles in a diploid state.

## Related tools

- **LRTK-SIM** (Simulates 10x Linked-Read data following the Chromium workflow, sweeping CF, CR, Mu_F and other config parameters to generate realistic test libraries at constant total coverage) — https://github.com/zhanglu295/LRTK-SIM
- **gen_fasta.py** (Auxiliary LRTK-SIM program that generates two haploid fasta source genomes by inserting SNVs from a VCF into the reference sequence) — https://github.com/zhanglu295/LRTK-SIM
- **Supernova2** (De novo assembler used to build diploid assemblies from each simulated or real 10x library, from which contig/scaffold contiguity metrics are extracted)
- **Evaluate_diploid_assembly** (Jupyter-based code for evaluating and comparing diploid assemblies produced under different parameters) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work; python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- Contig N50 decreases monotonically (or near-monotonically) as Mu_F increases at constant C = 56x, in both simulated and real libraries
- Contig N50 decreases as CF increases; assembly quality is optimal for CF between ~332x and 823x and degrades above 1,000x
- Length-weighted fragment length W_mu_FL lands in the ~50-150 kb optimal range for the best assemblies
- With broadly optimal library parameters, ~80% of the genome assembles in a diploid state in the Supernova2 output
- Simulated fastq outputs pass Supernova/Long Ranger validation: each library folder lib_X contains X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz

## Limitations

- The detrimental effect of extremely long fragments is only a proposed explanation (loss of barcode specificity for fragments spanning repetitive sequences) and is not verified mechanistically
- No alternative assembler to Supernova2 was available for comparison, so fragment-length effects on contiguity may be implementation-specific
- gen_fasta.py (this version) supports only SNVs when constructing diploid references; NA50-based accuracy comparison requires a high-quality human reference genome, which most non-human projects lack

## Evidence

- [abstract] Long DNA fragments could significantly extend phase blocks but decreased contig contiguity.: "Long DNA fragments could significantly extend phase blocks but decreased contig contiguity."
- [abstract] Optimal physical coverage window: "The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C."
- [readme] gen_fasta.py usage: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [readme] LRTK-SIM invocation: "Multiple libraries are allowed to simulated simutaneously and the parameters should be written in the correspinding configuration files."
- [discussion] in practice CF should be between 335x and 823x: [evidence span withheld — anchor status: not_found]
