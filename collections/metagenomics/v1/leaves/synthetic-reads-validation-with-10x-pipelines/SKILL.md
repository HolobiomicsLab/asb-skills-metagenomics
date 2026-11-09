---
name: synthetic-reads-validation-with-10x-pipelines
description: Use when simulated 10x Chromium linked-read FASTQs (gzipped R1/R2 pairs
  from LRTK-SIM) must be checked for compatibility with 10x official pipelines such
  as Long Ranger or Supernova2 before assembly or benchmarking.
when_to_use_negative:
- Input data are real 10x Chromium reads from a sequencer rather than LRTK-SIM synthetic
  output — pipeline compatibility is then a given, not something to validate
- The goal is simulating data for platforms other than 10x linked-reads (e.g. PacBio,
  ONT), which LRTK-SIM does not model
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3361
tools:
- name: LRTK-SIM
  role: Generates the barcoded Illumina paired-end FASTQ outputs whose 10x-pipeline
    compatibility this skill validates; accepts config files and writes lib_X folders
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Supernova2
  role: 10x official de novo assembler used as the acceptance check for the simulated
    FASTQ files
- name: Long Ranger
  role: 10x official genome/exome pipeline used as an acceptance check for the simulated
    FASTQ files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:cfa3c14c0e0210a5264836c051b5f7e877bec64fed2fa4f8c08ba5556233150b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - LRTK-SIM
  - Long Ranger
  - Supernova2
  - gen_fasta.py
  - multiprocessing
  - numpy
  - python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# synthetic-reads-validation-with-10x-pipelines

## Summary

Validates that synthetic 10x Chromium linked-read FASTQ output from LRTK-SIM is correctly formatted and accepted by the official 10x pipelines Long Ranger and Supernova, which the article reports as the acceptance criterion for realistic simulated Linked-Read data.

## When to use

Use when simulated 10x Chromium linked-read FASTQs (gzipped R1/R2 pairs from LRTK-SIM) must be checked for compatibility with 10x official pipelines such as Long Ranger or Supernova2 before assembly or benchmarking.

## When NOT to use

- Input data are real 10x Chromium reads from a sequencer rather than LRTK-SIM synthetic output — pipeline compatibility is then a given, not something to validate
- The goal is simulating data for platforms other than 10x linked-reads (e.g. PacBio, ONT), which LRTK-SIM does not model

## Inputs

- LRTK-SIM output folder lib_configX containing paired gzipped FASTQ files (configX_S1_L001_R1_001.fastq.gz and configX_S1_L001_R2_001.fastq.gz)
- LRTK-SIM configuration file (config1.txt/config2.txt) with parameter lines 2-33 (Path_Fastahap1/2, processors, CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, Hap)

## Outputs

- Confirmation that each library folder contains the expected paired gzipped FASTQ files for forward and reverse reads
- Evidence that the simulated reads are processable by 10x official pipelines Long Ranger and Supernova (acceptance check)

## How to apply

Run LRTK-SIM over its configuration folder (e.g. python LRTK-SIM.py ./diploid_config) with config files specifying haploid FASTA paths (line2/line3), CF (line7), CR (line9), N_FP (line11), Mu_F (line13), SR (line15), Error_rate (line21), Mu_IS/Std_IS (line27/line29), and Hap=2 for diploid simulation. Confirm each library folder lib_X contains the paired gzipped outputs X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz representing forward and reverse reads. The acceptance check is that the simulated data are formatted so they can be processed by the 10x official pipelines Long Ranger and Supernova2, as reported for LRTK-SIM output in the article.

## Related tools

- **LRTK-SIM** (Generates the barcoded Illumina paired-end FASTQ outputs whose 10x-pipeline compatibility this skill validates; accepts config files and writes lib_X folders) — https://github.com/zhanglu295/LRTK-SIM
- **Supernova2** (10x official de novo assembler used as the acceptance check for the simulated FASTQ files)
- **Long Ranger** (10x official genome/exome pipeline used as an acceptance check for the simulated FASTQ files)

## Examples

```
python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- Each library folder lib_X contains exactly two gzipped FASTQ files starting with X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz for forward and reverse reads
- The FASTQ read pairs carry 10x barcodes and are accepted by Long Ranger and Supernova without format errors
- Config file lines match the expected layout: line2/line3 haploid FASTA paths, line7 CF, line9 CR, line33 Hap=2 for diploid simulation
- The simulation ran over the correct configuration folder (python LRTK-SIM.py ./diploid_config) and emitted lib_config1/ and lib_config2/ when two libraries were simulated

## Limitations

- Only SNVs are supported for diploid reference generation in this version of gen_fasta.py; other variant types are not inserted
- Acceptance is reported for Long Ranger and Supernova specifically; compatibility with other tools is not asserted
- LRTK-SIM output has been tested and accepted by the 10x official pipelines, but no alternative assembler benchmark is implied (the article notes no other assembler could be compared to Supernova2)

## Evidence

- [readme] Acceptance of simulated data by 10x official pipelines: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger] and [Supernova]."
- [readme] Output file naming convention: "Two gzipped fastq files are generated and started by X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz to represent forward and reverse reads."
- [abstract] Purpose of LRTK-SIM in the article: "We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets."
- [methods] Config-file-driven invocation of the simulator: "python LRTK-SIM.py <path to configuration files>` Multiple libraries are allowed to simulated simutaneously"
