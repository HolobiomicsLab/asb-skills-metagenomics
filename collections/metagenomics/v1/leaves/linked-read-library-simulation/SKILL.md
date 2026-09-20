---
name: linked-read-library-simulation
description: Use when you need synthetic 10x Linked-Read FASTQs with controlled CF,
  CR, Mu_F, insert size, and error rate to benchmark diploid assemblers (e.g., Supernova2)
  or test assembly-parameter hypotheses against a gold standard.
when_to_use_negative:
- 'Haploid simulation with structural variants: gen_fasta.py in this version inserts
  SNVs only, so SV-heavy diploid references require a different generation route'
- 'Metagenomics simulation requiring species abundances: use LRTK-SIM''s dedicated
  meta_config route with an abundance flat file rather than the diploid config workflow'
- Evaluation of real (non-simulated) Linked-Read data where no gold standard or controlled
  parameter sweep is needed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: LRTK-SIM
  role: Simulates 10x Chromium Linked-Read libraries from configuration parameters,
    producing gzipped paired-end FASTQs accepted by 10x official pipelines
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: gen_fasta.py
  role: Auxiliary LRTK-SIM program that generates two haploid FASTA files by inserting
    SNVs from a VCF into the reference genome
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Supernova2
  role: Performs de novo diploid assembly of the simulated FASTQs so contiguity (N50)
    and accuracy (NA50) can be measured per parameter combination
- name: Evaluate_diploid_assembly
  role: Jupyter notebook workflow for evaluating and comparing the diploid assemblies
    produced from different simulated parameters
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:9884ec4e9e1b4ec5388808f07e962ff89ad6c54d9bb40239277fd1286f471475
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - 10x Linked-Read sequencing
  - Evaluate_diploid_assembly
  - LRTK-SIM
  - Python
  - Supernova2
  - gen_fasta.py
  - multiprocessing
  - numpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# linked-read-library-simulation

## Summary

Simulate realistic 10x Chromium Linked-Read libraries with LRTK-SIM by parameterizing long-fragment coverage (CF), per-fragment read coverage (CR), fragment length (Mu_F), and diploid haplotype inputs, to explore how library parameters affect de novo diploid assembly quality with Supernova2.

## When to use

Use when you need synthetic 10x Linked-Read FASTQs with controlled CF, CR, Mu_F, insert size, and error rate to benchmark diploid assemblers (e.g., Supernova2) or test assembly-parameter hypotheses against a gold standard.

## When NOT to use

- Haploid simulation with structural variants: gen_fasta.py in this version inserts SNVs only, so SV-heavy diploid references require a different generation route
- Metagenomics simulation requiring species abundances: use LRTK-SIM's dedicated meta_config route with an abundance flat file rather than the diploid config workflow
- Evaluation of real (non-simulated) Linked-Read data where no gold standard or controlled parameter sweep is needed

## Inputs

- Compressed or uncompressed VCF of variants for the sample
- Reference genome FASTA (e.g., hg38)
- LRTK-SIM configuration file(s) with Path_Fastahap1/Path_Fastahap2, processors, CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, Hap

## Outputs

- One lib_X folder per library containing X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz (forward/reverse reads)
- Two haploid reference FASTA files produced by gen_fasta.py for diploid simulation

## How to apply

First generate two haploid reference FASTAs by inserting SNVs from a VCF into the hg38 reference with gen_fasta.py (this version supports SNVs only), then point Path_Fastahap1/Path_Fastahap2 at them and set Hap=2 for diploid simulation. Author one LRTK-SIM configuration file per library in a config folder, setting CF (line7), CR (line9), N_FP, Mu_F (average fragment length in Kb), SR, Error_rate, Mu_IS, Std_IS, and processors. Run all libraries in parallel via 'python LRTK-SIM.py <path to configuration files>', which writes each library to a lib_X folder containing gzipped paired FASTQs (X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz). Sweep parameters to answer design questions — e.g., vary CF and CR at constant total coverage C = CR × CF (optimal CF 332x–823x, quality worsens >1,000x) or vary Mu_F at fixed C — then assemble each library with Supernova2 and evaluate contig/scaffold N50 and NA50.

## Related tools

- **LRTK-SIM** (Simulates 10x Chromium Linked-Read libraries from configuration parameters, producing gzipped paired-end FASTQs accepted by 10x official pipelines) — https://github.com/zhanglu295/LRTK-SIM
- **gen_fasta.py** (Auxiliary LRTK-SIM program that generates two haploid FASTA files by inserting SNVs from a VCF into the reference genome) — https://github.com/zhanglu295/LRTK-SIM
- **Supernova2** (Performs de novo diploid assembly of the simulated FASTQs so contiguity (N50) and accuracy (NA50) can be measured per parameter combination)
- **Evaluate_diploid_assembly** (Jupyter notebook workflow for evaluating and comparing the diploid assemblies produced from different simulated parameters) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work; python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- Every lib_X output folder contains both gzipped FASTQ files in the 10x naming format (X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz)
- Simulated reads are accepted by the 10x Long Ranger and Supernova official pipelines without rejection
- At constant total coverage C = 56x, assembly quality is optimal when CF falls between 332x and 823x and worsens when CF exceeds 1,000x
- Contig N50 decreases with increasing Mu_F at fixed C, matching the reported fragment-length effect, and the optimal length-weighted fragment length (W_mu_FL) is ~50-150 kb
- Sweeping total coverage C from 16x to 78x yields monotonically increasing contig N50, reproducing the reported shift from 621.4 to 758.1 kb beyond 56x

## Limitations

- gen_fasta.py in this version supports SNVs only; structural variants cannot be inserted into the diploid reference
- Generally only ~20-40% of the original long fragment sequence can be captured by the 10x Chromium partition amplification, which the simulator follows
- The 10x parameter space conclusions (e.g., optimal CF 332x-823x) are specific to Supernova2; without an alternative assembler the fragment-length effect cannot be reasoned about further
- LRTK-SIM requires python3 with sys, multiprocessing, numpy, os, gzip, and collections preinstalled

## Evidence

- [abstract] We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets.: "We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets."
- [readme] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [methods] python LRTK-SIM.py <path to configuration files> - multiple libraries simulated simultaneously: "`python LRTK-SIM.py <path to configuration files>`

Multiple libraries are allowed to simulated simutaneously"
- [readme] LRTK-SIM generates one folder per library named lib_X, with X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz, tested and accepted by 10x official pipelines.: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger]"
- [abstract] The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C.: "The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C."
